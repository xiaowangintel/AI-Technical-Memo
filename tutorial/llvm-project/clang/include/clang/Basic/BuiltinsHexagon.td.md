# BuiltinsHexagon.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/include/clang/Basic/BuiltinsHexagon.td`
- **Repository / 仓库**: `/root/xw/llvm-project`
- **Purpose (EN)**: Hexagon Builtin function defs *- C++.
- **Purpose (CN)**: 声明与 `BuiltinsHexagon` 相关的编译器基础元数据、诊断信息、目标描述或 builtin 定义。
- **Line Count / 行数**: 2214

## Line-by-Line Analysis / 逐行分析

### Lines 1-28

````tablegen
//===--- BuiltinsHexagon.td - Hexagon Builtin function defs -----*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines the Hexagon-specific builtin function database.
//
//===----------------------------------------------------------------------===//

include "clang/Basic/BuiltinsBase.td"

class VFeatures {
  string Features;
}

class V<string version, VFeatures newer> : VFeatures {
  let Features = !strconcat("v", version, "|", newer.Features);
}

let Features = "v81" in def V81 : VFeatures;

def V79 : V<"79", V81>;
def V75 : V<"75", V79>;
def V73 : V<"73", V75>;
def V71 : V<"71", V73>;
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
- **L9 EN**: Comment explains nearby logic, constraints, or intent: `This file defines the Hexagon-specific builtin function database.`.
  **L9 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This file defines the Hexagon-specific builtin function database.`。
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
- **L15 EN**: Declares TableGen class record `VFeatures`.
  **L15 CN**: 声明 TableGen class 记录 `VFeatures`。
- **L16 EN**: Adds a standalone statement or declaration: `string Features;`.
  **L16 CN**: 添加一条独立语句或声明：`string Features;`。
- **L17 EN**: Closes the current lexical scope or compound statement.
  **L17 CN**: 结束当前词法作用域或复合语句块。
- **L18 EN**: Blank line separating nearby declarations or logic blocks.
  **L18 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L19 EN**: Declares TableGen class record `V`.
  **L19 CN**: 声明 TableGen class 记录 `V`。
- **L20 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = !strconcat("v", version, "|", newer.Features);`.
  **L20 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = !strconcat("v", version, "|", newer.Features);`。
- **L21 EN**: Closes the current lexical scope or compound statement.
  **L21 CN**: 结束当前词法作用域或复合语句块。
- **L22 EN**: Blank line separating nearby declarations or logic blocks.
  **L22 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L23 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "v81" in def V81 : VFeatures;`.
  **L23 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "v81" in def V81 : VFeatures;`。
- **L24 EN**: Blank line separating nearby declarations or logic blocks.
  **L24 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L25 EN**: Declares TableGen def record `V79`.
  **L25 CN**: 声明 TableGen def 记录 `V79`。
- **L26 EN**: Declares TableGen def record `V75`.
  **L26 CN**: 声明 TableGen def 记录 `V75`。
- **L27 EN**: Declares TableGen def record `V73`.
  **L27 CN**: 声明 TableGen def 记录 `V73`。
- **L28 EN**: Declares TableGen def record `V71`.
  **L28 CN**: 声明 TableGen def 记录 `V71`。

### Lines 29-56

````tablegen
def V69 : V<"69", V71>;
def V68 : V<"68", V69>;
def V67 : V<"67", V68>;
def V66 : V<"66", V67>;
def V65 : V<"65", V66>;
def V62 : V<"62", V65>;
def V60 : V<"60", V62>;
def V55 : V<"55", V60>;
def V5  : V<"5", V55>;

class HVXVFeatures {
  string Features;
}

class HVXV<string version, HVXVFeatures newer> : HVXVFeatures {
  let Features = !strconcat("hvxv", version, "|", newer.Features);
}

let Features = "hvxv81" in def HVXV81 : HVXVFeatures;

def HVXV79 : HVXV<"79", HVXV81>;
def HVXV75 : HVXV<"75", HVXV79>;
def HVXV73 : HVXV<"73", HVXV75>;
def HVXV71 : HVXV<"71", HVXV73>;
def HVXV69 : HVXV<"69", HVXV71>;
def HVXV68 : HVXV<"68", HVXV69>;
def HVXV67 : HVXV<"67", HVXV68>;
def HVXV66 : HVXV<"66", HVXV67>;
````
- **L29 EN**: Declares TableGen def record `V69`.
  **L29 CN**: 声明 TableGen def 记录 `V69`。
- **L30 EN**: Declares TableGen def record `V68`.
  **L30 CN**: 声明 TableGen def 记录 `V68`。
- **L31 EN**: Declares TableGen def record `V67`.
  **L31 CN**: 声明 TableGen def 记录 `V67`。
- **L32 EN**: Declares TableGen def record `V66`.
  **L32 CN**: 声明 TableGen def 记录 `V66`。
- **L33 EN**: Declares TableGen def record `V65`.
  **L33 CN**: 声明 TableGen def 记录 `V65`。
- **L34 EN**: Declares TableGen def record `V62`.
  **L34 CN**: 声明 TableGen def 记录 `V62`。
- **L35 EN**: Declares TableGen def record `V60`.
  **L35 CN**: 声明 TableGen def 记录 `V60`。
- **L36 EN**: Declares TableGen def record `V55`.
  **L36 CN**: 声明 TableGen def 记录 `V55`。
- **L37 EN**: Declares TableGen def record `V5`.
  **L37 CN**: 声明 TableGen def 记录 `V5`。
- **L38 EN**: Blank line separating nearby declarations or logic blocks.
  **L38 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L39 EN**: Declares TableGen class record `HVXVFeatures`.
  **L39 CN**: 声明 TableGen class 记录 `HVXVFeatures`。
- **L40 EN**: Adds a standalone statement or declaration: `string Features;`.
  **L40 CN**: 添加一条独立语句或声明：`string Features;`。
- **L41 EN**: Closes the current lexical scope or compound statement.
  **L41 CN**: 结束当前词法作用域或复合语句块。
- **L42 EN**: Blank line separating nearby declarations or logic blocks.
  **L42 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L43 EN**: Declares TableGen class record `HVXV`.
  **L43 CN**: 声明 TableGen class 记录 `HVXV`。
- **L44 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = !strconcat("hvxv", version, "|", newer.Features);`.
  **L44 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = !strconcat("hvxv", version, "|", newer.Features);`。
- **L45 EN**: Closes the current lexical scope or compound statement.
  **L45 CN**: 结束当前词法作用域或复合语句块。
- **L46 EN**: Blank line separating nearby declarations or logic blocks.
  **L46 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L47 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "hvxv81" in def HVXV81 : HVXVFeatures;`.
  **L47 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "hvxv81" in def HVXV81 : HVXVFeatures;`。
- **L48 EN**: Blank line separating nearby declarations or logic blocks.
  **L48 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L49 EN**: Declares TableGen def record `HVXV79`.
  **L49 CN**: 声明 TableGen def 记录 `HVXV79`。
- **L50 EN**: Declares TableGen def record `HVXV75`.
  **L50 CN**: 声明 TableGen def 记录 `HVXV75`。
- **L51 EN**: Declares TableGen def record `HVXV73`.
  **L51 CN**: 声明 TableGen def 记录 `HVXV73`。
- **L52 EN**: Declares TableGen def record `HVXV71`.
  **L52 CN**: 声明 TableGen def 记录 `HVXV71`。
- **L53 EN**: Declares TableGen def record `HVXV69`.
  **L53 CN**: 声明 TableGen def 记录 `HVXV69`。
- **L54 EN**: Declares TableGen def record `HVXV68`.
  **L54 CN**: 声明 TableGen def 记录 `HVXV68`。
- **L55 EN**: Declares TableGen def record `HVXV67`.
  **L55 CN**: 声明 TableGen def 记录 `HVXV67`。
- **L56 EN**: Declares TableGen def record `HVXV66`.
  **L56 CN**: 声明 TableGen def 记录 `HVXV66`。

### Lines 57-84

````tablegen
def HVXV65 : HVXV<"65", HVXV66>;
def HVXV62 : HVXV<"62", HVXV65>;
def HVXV60 : HVXV<"60", HVXV62>;

def HexagonPrefix : NamePrefix<"__builtin_HEXAGON_">;

class HexagonBuiltin<string prototype> : TargetBuiltin {
  let Spellings = [NAME];
  let Prototype = prototype;
  let Features = V5.Features;
  let RequiredNamePrefix = HexagonPrefix; // Adds a prefix to the name.
}

class HexagonBuiltinNoPrefix<string prototype> : TargetBuiltin {
  let Spellings = [NAME];
  let Prototype = prototype;
  let Features = V5.Features;
}

// The builtins below are not autogenerated from iset.py.
// Make sure you do not overwrite these.
def __builtin_SI_to_SXTHI_asrh : HexagonBuiltinNoPrefix<"int(int)">;
def __builtin_brev_ldd : HexagonBuiltinNoPrefix<"void *(long long int * const, long long int *, int const)">;
def __builtin_brev_ldw : HexagonBuiltinNoPrefix<"void *(int * const, int *, int const)">;
def __builtin_brev_ldh : HexagonBuiltinNoPrefix<"void *(short * const, short *, int const)">;
def __builtin_brev_lduh : HexagonBuiltinNoPrefix<"void *(unsigned short * const, unsigned short *, int const)">;
def __builtin_brev_ldb : HexagonBuiltinNoPrefix<"void *(signed char * const, signed char *, int const)">;
def __builtin_brev_ldub : HexagonBuiltinNoPrefix<"void *(unsigned char * const, unsigned char *, int const)">;
````
- **L57 EN**: Declares TableGen def record `HVXV65`.
  **L57 CN**: 声明 TableGen def 记录 `HVXV65`。
- **L58 EN**: Declares TableGen def record `HVXV62`.
  **L58 CN**: 声明 TableGen def 记录 `HVXV62`。
- **L59 EN**: Declares TableGen def record `HVXV60`.
  **L59 CN**: 声明 TableGen def 记录 `HVXV60`。
- **L60 EN**: Blank line separating nearby declarations or logic blocks.
  **L60 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L61 EN**: Declares TableGen def record `HexagonPrefix`.
  **L61 CN**: 声明 TableGen def 记录 `HexagonPrefix`。
- **L62 EN**: Blank line separating nearby declarations or logic blocks.
  **L62 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L63 EN**: Declares TableGen class record `HexagonBuiltin`.
  **L63 CN**: 声明 TableGen class 记录 `HexagonBuiltin`。
- **L64 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Spellings = [NAME];`.
  **L64 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Spellings = [NAME];`。
- **L65 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Prototype = prototype;`.
  **L65 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Prototype = prototype;`。
- **L66 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = V5.Features;`.
  **L66 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = V5.Features;`。
- **L67 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredNamePrefix = HexagonPrefix; // Adds a prefix to the name.`.
  **L67 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredNamePrefix = HexagonPrefix; // Adds a prefix to the name.`。
- **L68 EN**: Closes the current lexical scope or compound statement.
  **L68 CN**: 结束当前词法作用域或复合语句块。
- **L69 EN**: Blank line separating nearby declarations or logic blocks.
  **L69 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L70 EN**: Declares TableGen class record `HexagonBuiltinNoPrefix`.
  **L70 CN**: 声明 TableGen class 记录 `HexagonBuiltinNoPrefix`。
- **L71 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Spellings = [NAME];`.
  **L71 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Spellings = [NAME];`。
- **L72 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Prototype = prototype;`.
  **L72 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Prototype = prototype;`。
- **L73 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = V5.Features;`.
  **L73 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = V5.Features;`。
- **L74 EN**: Closes the current lexical scope or compound statement.
  **L74 CN**: 结束当前词法作用域或复合语句块。
- **L75 EN**: Blank line separating nearby declarations or logic blocks.
  **L75 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L76 EN**: Comment explains nearby logic, constraints, or intent: `The builtins below are not autogenerated from iset.py.`.
  **L76 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The builtins below are not autogenerated from iset.py.`。
- **L77 EN**: Comment explains nearby logic, constraints, or intent: `Make sure you do not overwrite these.`.
  **L77 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Make sure you do not overwrite these.`。
- **L78 EN**: Declares TableGen def record `__builtin_SI_to_SXTHI_asrh`.
  **L78 CN**: 声明 TableGen def 记录 `__builtin_SI_to_SXTHI_asrh`。
- **L79 EN**: Declares TableGen def record `__builtin_brev_ldd`.
  **L79 CN**: 声明 TableGen def 记录 `__builtin_brev_ldd`。
- **L80 EN**: Declares TableGen def record `__builtin_brev_ldw`.
  **L80 CN**: 声明 TableGen def 记录 `__builtin_brev_ldw`。
- **L81 EN**: Declares TableGen def record `__builtin_brev_ldh`.
  **L81 CN**: 声明 TableGen def 记录 `__builtin_brev_ldh`。
- **L82 EN**: Declares TableGen def record `__builtin_brev_lduh`.
  **L82 CN**: 声明 TableGen def 记录 `__builtin_brev_lduh`。
- **L83 EN**: Declares TableGen def record `__builtin_brev_ldb`.
  **L83 CN**: 声明 TableGen def 记录 `__builtin_brev_ldb`。
- **L84 EN**: Declares TableGen def record `__builtin_brev_ldub`.
  **L84 CN**: 声明 TableGen def 记录 `__builtin_brev_ldub`。

### Lines 85-112

````tablegen
def __builtin_circ_ldd : HexagonBuiltinNoPrefix<"long long int *(long long int *, long long int *, int, _Constant int)">;
def __builtin_circ_ldw : HexagonBuiltinNoPrefix<"int *(int *, int *, int, _Constant int)">;
def __builtin_circ_ldh : HexagonBuiltinNoPrefix<"short *(short *, short *, int, _Constant int)">;
def __builtin_circ_lduh : HexagonBuiltinNoPrefix<"unsigned short *(unsigned short *, unsigned short *, int, _Constant int)">;
def __builtin_circ_ldb : HexagonBuiltinNoPrefix<"char *(char *, char *, int, _Constant int)">;
def __builtin_circ_ldub : HexagonBuiltinNoPrefix<"unsigned char *(unsigned char *, unsigned char *, int, _Constant int)">;
def __builtin_brev_std : HexagonBuiltinNoPrefix<"long long int * const(long long int *, long long int, int const)">;
def __builtin_brev_stw : HexagonBuiltinNoPrefix<"int * const(int *, int, int const)">;
def __builtin_brev_sth : HexagonBuiltinNoPrefix<"short * const(short *, int, int const)">;
def __builtin_brev_sthhi : HexagonBuiltinNoPrefix<"short * const(short *, int, int const)">;
def __builtin_brev_stb : HexagonBuiltinNoPrefix<"char * const(char *, int, int const)">;
def __builtin_circ_std : HexagonBuiltinNoPrefix<"long long int *(long long int *, long long int, int, _Constant int)">;
def __builtin_circ_stw : HexagonBuiltinNoPrefix<"int *(int *, int, int, _Constant int)">;
def __builtin_circ_sth : HexagonBuiltinNoPrefix<"short *(short *, int, int, _Constant int)">;
def __builtin_circ_sthhi : HexagonBuiltinNoPrefix<"short *(short *, int, int, _Constant int)">;
def __builtin_circ_stb : HexagonBuiltinNoPrefix<"char *(char *, int, int, _Constant int)">;
def L2_loadrub_pci : HexagonBuiltin<"int(void *, _Constant int, int, void const *)">;
def L2_loadrb_pci : HexagonBuiltin<"int(void *, _Constant int, int, void const *)">;
def L2_loadruh_pci : HexagonBuiltin<"int(void *, _Constant int, int, void const *)">;
def L2_loadrh_pci : HexagonBuiltin<"int(void *, _Constant int, int, void const *)">;
def L2_loadri_pci : HexagonBuiltin<"int(void *, _Constant int, int, void const *)">;
def L2_loadrd_pci : HexagonBuiltin<"long long int(void *, _Constant int, int, void const *)">;
def L2_loadrub_pcr : HexagonBuiltin<"int(void *, int, void const *)">;
def L2_loadrb_pcr : HexagonBuiltin<"int(void *, int, void const *)">;
def L2_loadruh_pcr : HexagonBuiltin<"int(void *, int, void const *)">;
def L2_loadrh_pcr : HexagonBuiltin<"int(void *, int, void const *)">;
def L2_loadri_pcr : HexagonBuiltin<"int(void *, int, void const *)">;
def L2_loadrd_pcr : HexagonBuiltin<"long long int(void *, int, void const *)">;
````
- **L85 EN**: Declares TableGen def record `__builtin_circ_ldd`.
  **L85 CN**: 声明 TableGen def 记录 `__builtin_circ_ldd`。
- **L86 EN**: Declares TableGen def record `__builtin_circ_ldw`.
  **L86 CN**: 声明 TableGen def 记录 `__builtin_circ_ldw`。
- **L87 EN**: Declares TableGen def record `__builtin_circ_ldh`.
  **L87 CN**: 声明 TableGen def 记录 `__builtin_circ_ldh`。
- **L88 EN**: Declares TableGen def record `__builtin_circ_lduh`.
  **L88 CN**: 声明 TableGen def 记录 `__builtin_circ_lduh`。
- **L89 EN**: Declares TableGen def record `__builtin_circ_ldb`.
  **L89 CN**: 声明 TableGen def 记录 `__builtin_circ_ldb`。
- **L90 EN**: Declares TableGen def record `__builtin_circ_ldub`.
  **L90 CN**: 声明 TableGen def 记录 `__builtin_circ_ldub`。
- **L91 EN**: Declares TableGen def record `__builtin_brev_std`.
  **L91 CN**: 声明 TableGen def 记录 `__builtin_brev_std`。
- **L92 EN**: Declares TableGen def record `__builtin_brev_stw`.
  **L92 CN**: 声明 TableGen def 记录 `__builtin_brev_stw`。
- **L93 EN**: Declares TableGen def record `__builtin_brev_sth`.
  **L93 CN**: 声明 TableGen def 记录 `__builtin_brev_sth`。
- **L94 EN**: Declares TableGen def record `__builtin_brev_sthhi`.
  **L94 CN**: 声明 TableGen def 记录 `__builtin_brev_sthhi`。
- **L95 EN**: Declares TableGen def record `__builtin_brev_stb`.
  **L95 CN**: 声明 TableGen def 记录 `__builtin_brev_stb`。
- **L96 EN**: Declares TableGen def record `__builtin_circ_std`.
  **L96 CN**: 声明 TableGen def 记录 `__builtin_circ_std`。
- **L97 EN**: Declares TableGen def record `__builtin_circ_stw`.
  **L97 CN**: 声明 TableGen def 记录 `__builtin_circ_stw`。
- **L98 EN**: Declares TableGen def record `__builtin_circ_sth`.
  **L98 CN**: 声明 TableGen def 记录 `__builtin_circ_sth`。
- **L99 EN**: Declares TableGen def record `__builtin_circ_sthhi`.
  **L99 CN**: 声明 TableGen def 记录 `__builtin_circ_sthhi`。
- **L100 EN**: Declares TableGen def record `__builtin_circ_stb`.
  **L100 CN**: 声明 TableGen def 记录 `__builtin_circ_stb`。
- **L101 EN**: Declares TableGen def record `L2_loadrub_pci`.
  **L101 CN**: 声明 TableGen def 记录 `L2_loadrub_pci`。
- **L102 EN**: Declares TableGen def record `L2_loadrb_pci`.
  **L102 CN**: 声明 TableGen def 记录 `L2_loadrb_pci`。
- **L103 EN**: Declares TableGen def record `L2_loadruh_pci`.
  **L103 CN**: 声明 TableGen def 记录 `L2_loadruh_pci`。
- **L104 EN**: Declares TableGen def record `L2_loadrh_pci`.
  **L104 CN**: 声明 TableGen def 记录 `L2_loadrh_pci`。
- **L105 EN**: Declares TableGen def record `L2_loadri_pci`.
  **L105 CN**: 声明 TableGen def 记录 `L2_loadri_pci`。
- **L106 EN**: Declares TableGen def record `L2_loadrd_pci`.
  **L106 CN**: 声明 TableGen def 记录 `L2_loadrd_pci`。
- **L107 EN**: Declares TableGen def record `L2_loadrub_pcr`.
  **L107 CN**: 声明 TableGen def 记录 `L2_loadrub_pcr`。
- **L108 EN**: Declares TableGen def record `L2_loadrb_pcr`.
  **L108 CN**: 声明 TableGen def 记录 `L2_loadrb_pcr`。
- **L109 EN**: Declares TableGen def record `L2_loadruh_pcr`.
  **L109 CN**: 声明 TableGen def 记录 `L2_loadruh_pcr`。
- **L110 EN**: Declares TableGen def record `L2_loadrh_pcr`.
  **L110 CN**: 声明 TableGen def 记录 `L2_loadrh_pcr`。
- **L111 EN**: Declares TableGen def record `L2_loadri_pcr`.
  **L111 CN**: 声明 TableGen def 记录 `L2_loadri_pcr`。
- **L112 EN**: Declares TableGen def record `L2_loadrd_pcr`.
  **L112 CN**: 声明 TableGen def 记录 `L2_loadrd_pcr`。

### Lines 113-140

````tablegen

def S2_storerb_pci : HexagonBuiltin<"void(void *, _Constant int, int, int, void const *)">;
def S2_storerh_pci : HexagonBuiltin<"void(void *, _Constant int, int, int, void const *)">;
def S2_storerf_pci : HexagonBuiltin<"void(void *, _Constant int, int, int, void const *)">;
def S2_storeri_pci : HexagonBuiltin<"void(void *, _Constant int, int, int, void const *)">;
def S2_storerd_pci : HexagonBuiltin<"void(void *, _Constant int, int, long long int, void const *)">;
def S2_storerb_pcr : HexagonBuiltin<"void(void *, int, int, void const *)">;
def S2_storerh_pcr : HexagonBuiltin<"void(void *, int, int, void const *)">;
def S2_storerf_pcr : HexagonBuiltin<"void(void *, int, int, void const *)">;
def S2_storeri_pcr : HexagonBuiltin<"void(void *, int, int, void const *)">;
def S2_storerd_pcr : HexagonBuiltin<"void(void *, int, long long int, void const *)">;

def prefetch : HexagonBuiltin<"void(void *)">;
let Features = V62.Features in {
  def A6_vminub_RdP : HexagonBuiltin<"long long int(long long int, long long int)">;
}

let Features = HVXV60.Features in {
  def V6_vmaskedstoreq : HexagonBuiltin<"void(_Vector<64, bool>, void *, _Vector<16, int>)">;
  def V6_vmaskedstorenq : HexagonBuiltin<"void(_Vector<64, bool>, void *, _Vector<16, int>)">;
  def V6_vmaskedstorentq : HexagonBuiltin<"void(_Vector<64, bool>, void *, _Vector<16, int>)">;
  def V6_vmaskedstorentnq : HexagonBuiltin<"void(_Vector<64, bool>, void *, _Vector<16, int>)">;
  def V6_vmaskedstoreq_128B : HexagonBuiltin<"void(_Vector<128, bool>, void *, _Vector<32, int>)">;
  def V6_vmaskedstorenq_128B : HexagonBuiltin<"void(_Vector<128, bool>, void *, _Vector<32, int>)">;
  def V6_vmaskedstorentq_128B : HexagonBuiltin<"void(_Vector<128, bool>, void *, _Vector<32, int>)">;
  def V6_vmaskedstorentnq_128B : HexagonBuiltin<"void(_Vector<128, bool>, void *, _Vector<32, int>)">;
}

````
- **L113 EN**: Blank line separating nearby declarations or logic blocks.
  **L113 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L114 EN**: Declares TableGen def record `S2_storerb_pci`.
  **L114 CN**: 声明 TableGen def 记录 `S2_storerb_pci`。
- **L115 EN**: Declares TableGen def record `S2_storerh_pci`.
  **L115 CN**: 声明 TableGen def 记录 `S2_storerh_pci`。
- **L116 EN**: Declares TableGen def record `S2_storerf_pci`.
  **L116 CN**: 声明 TableGen def 记录 `S2_storerf_pci`。
- **L117 EN**: Declares TableGen def record `S2_storeri_pci`.
  **L117 CN**: 声明 TableGen def 记录 `S2_storeri_pci`。
- **L118 EN**: Declares TableGen def record `S2_storerd_pci`.
  **L118 CN**: 声明 TableGen def 记录 `S2_storerd_pci`。
- **L119 EN**: Declares TableGen def record `S2_storerb_pcr`.
  **L119 CN**: 声明 TableGen def 记录 `S2_storerb_pcr`。
- **L120 EN**: Declares TableGen def record `S2_storerh_pcr`.
  **L120 CN**: 声明 TableGen def 记录 `S2_storerh_pcr`。
- **L121 EN**: Declares TableGen def record `S2_storerf_pcr`.
  **L121 CN**: 声明 TableGen def 记录 `S2_storerf_pcr`。
- **L122 EN**: Declares TableGen def record `S2_storeri_pcr`.
  **L122 CN**: 声明 TableGen def 记录 `S2_storeri_pcr`。
- **L123 EN**: Declares TableGen def record `S2_storerd_pcr`.
  **L123 CN**: 声明 TableGen def 记录 `S2_storerd_pcr`。
- **L124 EN**: Blank line separating nearby declarations or logic blocks.
  **L124 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L125 EN**: Declares TableGen def record `prefetch`.
  **L125 CN**: 声明 TableGen def 记录 `prefetch`。
- **L126 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = V62.Features in {`.
  **L126 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = V62.Features in {`。
- **L127 EN**: Declares TableGen def record `A6_vminub_RdP`.
  **L127 CN**: 声明 TableGen def 记录 `A6_vminub_RdP`。
- **L128 EN**: Closes the current lexical scope or compound statement.
  **L128 CN**: 结束当前词法作用域或复合语句块。
- **L129 EN**: Blank line separating nearby declarations or logic blocks.
  **L129 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L130 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = HVXV60.Features in {`.
  **L130 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = HVXV60.Features in {`。
- **L131 EN**: Declares TableGen def record `V6_vmaskedstoreq`.
  **L131 CN**: 声明 TableGen def 记录 `V6_vmaskedstoreq`。
- **L132 EN**: Declares TableGen def record `V6_vmaskedstorenq`.
  **L132 CN**: 声明 TableGen def 记录 `V6_vmaskedstorenq`。
- **L133 EN**: Declares TableGen def record `V6_vmaskedstorentq`.
  **L133 CN**: 声明 TableGen def 记录 `V6_vmaskedstorentq`。
- **L134 EN**: Declares TableGen def record `V6_vmaskedstorentnq`.
  **L134 CN**: 声明 TableGen def 记录 `V6_vmaskedstorentnq`。
- **L135 EN**: Declares TableGen def record `V6_vmaskedstoreq_128B`.
  **L135 CN**: 声明 TableGen def 记录 `V6_vmaskedstoreq_128B`。
- **L136 EN**: Declares TableGen def record `V6_vmaskedstorenq_128B`.
  **L136 CN**: 声明 TableGen def 记录 `V6_vmaskedstorenq_128B`。
- **L137 EN**: Declares TableGen def record `V6_vmaskedstorentq_128B`.
  **L137 CN**: 声明 TableGen def 记录 `V6_vmaskedstorentq_128B`。
- **L138 EN**: Declares TableGen def record `V6_vmaskedstorentnq_128B`.
  **L138 CN**: 声明 TableGen def 记录 `V6_vmaskedstorentnq_128B`。
- **L139 EN**: Closes the current lexical scope or compound statement.
  **L139 CN**: 结束当前词法作用域或复合语句块。
- **L140 EN**: Blank line separating nearby declarations or logic blocks.
  **L140 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 141-168

````tablegen

// These are only valid on v65
let Features = "hvxv65" in {
  def V6_vrmpybub_rtt : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>, long long int)">;
  def V6_vrmpybub_rtt_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>, long long int)">;
  def V6_vrmpybub_rtt_acc : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<16, int>, long long int)">;
  def V6_vrmpybub_rtt_acc_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<32, int>, long long int)">;
  def V6_vrmpyub_rtt : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>, long long int)">;
  def V6_vrmpyub_rtt_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>, long long int)">;
  def V6_vrmpyub_rtt_acc : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<16, int>, long long int)">;
  def V6_vrmpyub_rtt_acc_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<32, int>, long long int)">;
}

// V5 Scalar Instructions.

def A2_abs : HexagonBuiltin<"int(int)">;
def A2_absp : HexagonBuiltin<"long long int(long long int)">;
def A2_abssat : HexagonBuiltin<"int(int)">;
def A2_add : HexagonBuiltin<"int(int, int)">;
def A2_addh_h16_hh : HexagonBuiltin<"int(int, int)">;
def A2_addh_h16_hl : HexagonBuiltin<"int(int, int)">;
def A2_addh_h16_lh : HexagonBuiltin<"int(int, int)">;
def A2_addh_h16_ll : HexagonBuiltin<"int(int, int)">;
def A2_addh_h16_sat_hh : HexagonBuiltin<"int(int, int)">;
def A2_addh_h16_sat_hl : HexagonBuiltin<"int(int, int)">;
def A2_addh_h16_sat_lh : HexagonBuiltin<"int(int, int)">;
def A2_addh_h16_sat_ll : HexagonBuiltin<"int(int, int)">;
def A2_addh_l16_hl : HexagonBuiltin<"int(int, int)">;
````
- **L141 EN**: Blank line separating nearby declarations or logic blocks.
  **L141 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L142 EN**: Comment explains nearby logic, constraints, or intent: `These are only valid on v65`.
  **L142 CN**: 注释解释附近代码的逻辑、约束或设计意图：`These are only valid on v65`。
- **L143 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "hvxv65" in {`.
  **L143 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "hvxv65" in {`。
- **L144 EN**: Declares TableGen def record `V6_vrmpybub_rtt`.
  **L144 CN**: 声明 TableGen def 记录 `V6_vrmpybub_rtt`。
- **L145 EN**: Declares TableGen def record `V6_vrmpybub_rtt_128B`.
  **L145 CN**: 声明 TableGen def 记录 `V6_vrmpybub_rtt_128B`。
- **L146 EN**: Declares TableGen def record `V6_vrmpybub_rtt_acc`.
  **L146 CN**: 声明 TableGen def 记录 `V6_vrmpybub_rtt_acc`。
- **L147 EN**: Declares TableGen def record `V6_vrmpybub_rtt_acc_128B`.
  **L147 CN**: 声明 TableGen def 记录 `V6_vrmpybub_rtt_acc_128B`。
- **L148 EN**: Declares TableGen def record `V6_vrmpyub_rtt`.
  **L148 CN**: 声明 TableGen def 记录 `V6_vrmpyub_rtt`。
- **L149 EN**: Declares TableGen def record `V6_vrmpyub_rtt_128B`.
  **L149 CN**: 声明 TableGen def 记录 `V6_vrmpyub_rtt_128B`。
- **L150 EN**: Declares TableGen def record `V6_vrmpyub_rtt_acc`.
  **L150 CN**: 声明 TableGen def 记录 `V6_vrmpyub_rtt_acc`。
- **L151 EN**: Declares TableGen def record `V6_vrmpyub_rtt_acc_128B`.
  **L151 CN**: 声明 TableGen def 记录 `V6_vrmpyub_rtt_acc_128B`。
- **L152 EN**: Closes the current lexical scope or compound statement.
  **L152 CN**: 结束当前词法作用域或复合语句块。
- **L153 EN**: Blank line separating nearby declarations or logic blocks.
  **L153 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L154 EN**: Comment explains nearby logic, constraints, or intent: `V5 Scalar Instructions.`.
  **L154 CN**: 注释解释附近代码的逻辑、约束或设计意图：`V5 Scalar Instructions.`。
- **L155 EN**: Blank line separating nearby declarations or logic blocks.
  **L155 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L156 EN**: Declares TableGen def record `A2_abs`.
  **L156 CN**: 声明 TableGen def 记录 `A2_abs`。
- **L157 EN**: Declares TableGen def record `A2_absp`.
  **L157 CN**: 声明 TableGen def 记录 `A2_absp`。
- **L158 EN**: Declares TableGen def record `A2_abssat`.
  **L158 CN**: 声明 TableGen def 记录 `A2_abssat`。
- **L159 EN**: Declares TableGen def record `A2_add`.
  **L159 CN**: 声明 TableGen def 记录 `A2_add`。
- **L160 EN**: Declares TableGen def record `A2_addh_h16_hh`.
  **L160 CN**: 声明 TableGen def 记录 `A2_addh_h16_hh`。
- **L161 EN**: Declares TableGen def record `A2_addh_h16_hl`.
  **L161 CN**: 声明 TableGen def 记录 `A2_addh_h16_hl`。
- **L162 EN**: Declares TableGen def record `A2_addh_h16_lh`.
  **L162 CN**: 声明 TableGen def 记录 `A2_addh_h16_lh`。
- **L163 EN**: Declares TableGen def record `A2_addh_h16_ll`.
  **L163 CN**: 声明 TableGen def 记录 `A2_addh_h16_ll`。
- **L164 EN**: Declares TableGen def record `A2_addh_h16_sat_hh`.
  **L164 CN**: 声明 TableGen def 记录 `A2_addh_h16_sat_hh`。
- **L165 EN**: Declares TableGen def record `A2_addh_h16_sat_hl`.
  **L165 CN**: 声明 TableGen def 记录 `A2_addh_h16_sat_hl`。
- **L166 EN**: Declares TableGen def record `A2_addh_h16_sat_lh`.
  **L166 CN**: 声明 TableGen def 记录 `A2_addh_h16_sat_lh`。
- **L167 EN**: Declares TableGen def record `A2_addh_h16_sat_ll`.
  **L167 CN**: 声明 TableGen def 记录 `A2_addh_h16_sat_ll`。
- **L168 EN**: Declares TableGen def record `A2_addh_l16_hl`.
  **L168 CN**: 声明 TableGen def 记录 `A2_addh_l16_hl`。

### Lines 169-196

````tablegen
def A2_addh_l16_ll : HexagonBuiltin<"int(int, int)">;
def A2_addh_l16_sat_hl : HexagonBuiltin<"int(int, int)">;
def A2_addh_l16_sat_ll : HexagonBuiltin<"int(int, int)">;
def A2_addi : HexagonBuiltin<"int(int, _Constant int)">;
def A2_addp : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_addpsat : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_addsat : HexagonBuiltin<"int(int, int)">;
def A2_addsp : HexagonBuiltin<"long long int(int, long long int)">;
def A2_and : HexagonBuiltin<"int(int, int)">;
def A2_andir : HexagonBuiltin<"int(int, _Constant int)">;
def A2_andp : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_aslh : HexagonBuiltin<"int(int)">;
def A2_asrh : HexagonBuiltin<"int(int)">;
def A2_combine_hh : HexagonBuiltin<"int(int, int)">;
def A2_combine_hl : HexagonBuiltin<"int(int, int)">;
def A2_combine_lh : HexagonBuiltin<"int(int, int)">;
def A2_combine_ll : HexagonBuiltin<"int(int, int)">;
def A2_combineii : HexagonBuiltin<"long long int(_Constant int, _Constant int)">;
def A2_combinew : HexagonBuiltin<"long long int(int, int)">;
def A2_max : HexagonBuiltin<"int(int, int)">;
def A2_maxp : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_maxu : HexagonBuiltin<"unsigned int(int, int)">;
def A2_maxup : HexagonBuiltin<"unsigned long long int(long long int, long long int)">;
def A2_min : HexagonBuiltin<"int(int, int)">;
def A2_minp : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_minu : HexagonBuiltin<"unsigned int(int, int)">;
def A2_minup : HexagonBuiltin<"unsigned long long int(long long int, long long int)">;
def A2_neg : HexagonBuiltin<"int(int)">;
````
- **L169 EN**: Declares TableGen def record `A2_addh_l16_ll`.
  **L169 CN**: 声明 TableGen def 记录 `A2_addh_l16_ll`。
- **L170 EN**: Declares TableGen def record `A2_addh_l16_sat_hl`.
  **L170 CN**: 声明 TableGen def 记录 `A2_addh_l16_sat_hl`。
- **L171 EN**: Declares TableGen def record `A2_addh_l16_sat_ll`.
  **L171 CN**: 声明 TableGen def 记录 `A2_addh_l16_sat_ll`。
- **L172 EN**: Declares TableGen def record `A2_addi`.
  **L172 CN**: 声明 TableGen def 记录 `A2_addi`。
- **L173 EN**: Declares TableGen def record `A2_addp`.
  **L173 CN**: 声明 TableGen def 记录 `A2_addp`。
- **L174 EN**: Declares TableGen def record `A2_addpsat`.
  **L174 CN**: 声明 TableGen def 记录 `A2_addpsat`。
- **L175 EN**: Declares TableGen def record `A2_addsat`.
  **L175 CN**: 声明 TableGen def 记录 `A2_addsat`。
- **L176 EN**: Declares TableGen def record `A2_addsp`.
  **L176 CN**: 声明 TableGen def 记录 `A2_addsp`。
- **L177 EN**: Declares TableGen def record `A2_and`.
  **L177 CN**: 声明 TableGen def 记录 `A2_and`。
- **L178 EN**: Declares TableGen def record `A2_andir`.
  **L178 CN**: 声明 TableGen def 记录 `A2_andir`。
- **L179 EN**: Declares TableGen def record `A2_andp`.
  **L179 CN**: 声明 TableGen def 记录 `A2_andp`。
- **L180 EN**: Declares TableGen def record `A2_aslh`.
  **L180 CN**: 声明 TableGen def 记录 `A2_aslh`。
- **L181 EN**: Declares TableGen def record `A2_asrh`.
  **L181 CN**: 声明 TableGen def 记录 `A2_asrh`。
- **L182 EN**: Declares TableGen def record `A2_combine_hh`.
  **L182 CN**: 声明 TableGen def 记录 `A2_combine_hh`。
- **L183 EN**: Declares TableGen def record `A2_combine_hl`.
  **L183 CN**: 声明 TableGen def 记录 `A2_combine_hl`。
- **L184 EN**: Declares TableGen def record `A2_combine_lh`.
  **L184 CN**: 声明 TableGen def 记录 `A2_combine_lh`。
- **L185 EN**: Declares TableGen def record `A2_combine_ll`.
  **L185 CN**: 声明 TableGen def 记录 `A2_combine_ll`。
- **L186 EN**: Declares TableGen def record `A2_combineii`.
  **L186 CN**: 声明 TableGen def 记录 `A2_combineii`。
- **L187 EN**: Declares TableGen def record `A2_combinew`.
  **L187 CN**: 声明 TableGen def 记录 `A2_combinew`。
- **L188 EN**: Declares TableGen def record `A2_max`.
  **L188 CN**: 声明 TableGen def 记录 `A2_max`。
- **L189 EN**: Declares TableGen def record `A2_maxp`.
  **L189 CN**: 声明 TableGen def 记录 `A2_maxp`。
- **L190 EN**: Declares TableGen def record `A2_maxu`.
  **L190 CN**: 声明 TableGen def 记录 `A2_maxu`。
- **L191 EN**: Declares TableGen def record `A2_maxup`.
  **L191 CN**: 声明 TableGen def 记录 `A2_maxup`。
- **L192 EN**: Declares TableGen def record `A2_min`.
  **L192 CN**: 声明 TableGen def 记录 `A2_min`。
- **L193 EN**: Declares TableGen def record `A2_minp`.
  **L193 CN**: 声明 TableGen def 记录 `A2_minp`。
- **L194 EN**: Declares TableGen def record `A2_minu`.
  **L194 CN**: 声明 TableGen def 记录 `A2_minu`。
- **L195 EN**: Declares TableGen def record `A2_minup`.
  **L195 CN**: 声明 TableGen def 记录 `A2_minup`。
- **L196 EN**: Declares TableGen def record `A2_neg`.
  **L196 CN**: 声明 TableGen def 记录 `A2_neg`。

### Lines 197-224

````tablegen
def A2_negp : HexagonBuiltin<"long long int(long long int)">;
def A2_negsat : HexagonBuiltin<"int(int)">;
def A2_not : HexagonBuiltin<"int(int)">;
def A2_notp : HexagonBuiltin<"long long int(long long int)">;
def A2_or : HexagonBuiltin<"int(int, int)">;
def A2_orir : HexagonBuiltin<"int(int, _Constant int)">;
def A2_orp : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_roundsat : HexagonBuiltin<"int(long long int)">;
def A2_sat : HexagonBuiltin<"int(long long int)">;
def A2_satb : HexagonBuiltin<"int(int)">;
def A2_sath : HexagonBuiltin<"int(int)">;
def A2_satub : HexagonBuiltin<"int(int)">;
def A2_satuh : HexagonBuiltin<"int(int)">;
def A2_sub : HexagonBuiltin<"int(int, int)">;
def A2_subh_h16_hh : HexagonBuiltin<"int(int, int)">;
def A2_subh_h16_hl : HexagonBuiltin<"int(int, int)">;
def A2_subh_h16_lh : HexagonBuiltin<"int(int, int)">;
def A2_subh_h16_ll : HexagonBuiltin<"int(int, int)">;
def A2_subh_h16_sat_hh : HexagonBuiltin<"int(int, int)">;
def A2_subh_h16_sat_hl : HexagonBuiltin<"int(int, int)">;
def A2_subh_h16_sat_lh : HexagonBuiltin<"int(int, int)">;
def A2_subh_h16_sat_ll : HexagonBuiltin<"int(int, int)">;
def A2_subh_l16_hl : HexagonBuiltin<"int(int, int)">;
def A2_subh_l16_ll : HexagonBuiltin<"int(int, int)">;
def A2_subh_l16_sat_hl : HexagonBuiltin<"int(int, int)">;
def A2_subh_l16_sat_ll : HexagonBuiltin<"int(int, int)">;
def A2_subp : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_subri : HexagonBuiltin<"int(_Constant int, int)">;
````
- **L197 EN**: Declares TableGen def record `A2_negp`.
  **L197 CN**: 声明 TableGen def 记录 `A2_negp`。
- **L198 EN**: Declares TableGen def record `A2_negsat`.
  **L198 CN**: 声明 TableGen def 记录 `A2_negsat`。
- **L199 EN**: Declares TableGen def record `A2_not`.
  **L199 CN**: 声明 TableGen def 记录 `A2_not`。
- **L200 EN**: Declares TableGen def record `A2_notp`.
  **L200 CN**: 声明 TableGen def 记录 `A2_notp`。
- **L201 EN**: Declares TableGen def record `A2_or`.
  **L201 CN**: 声明 TableGen def 记录 `A2_or`。
- **L202 EN**: Declares TableGen def record `A2_orir`.
  **L202 CN**: 声明 TableGen def 记录 `A2_orir`。
- **L203 EN**: Declares TableGen def record `A2_orp`.
  **L203 CN**: 声明 TableGen def 记录 `A2_orp`。
- **L204 EN**: Declares TableGen def record `A2_roundsat`.
  **L204 CN**: 声明 TableGen def 记录 `A2_roundsat`。
- **L205 EN**: Declares TableGen def record `A2_sat`.
  **L205 CN**: 声明 TableGen def 记录 `A2_sat`。
- **L206 EN**: Declares TableGen def record `A2_satb`.
  **L206 CN**: 声明 TableGen def 记录 `A2_satb`。
- **L207 EN**: Declares TableGen def record `A2_sath`.
  **L207 CN**: 声明 TableGen def 记录 `A2_sath`。
- **L208 EN**: Declares TableGen def record `A2_satub`.
  **L208 CN**: 声明 TableGen def 记录 `A2_satub`。
- **L209 EN**: Declares TableGen def record `A2_satuh`.
  **L209 CN**: 声明 TableGen def 记录 `A2_satuh`。
- **L210 EN**: Declares TableGen def record `A2_sub`.
  **L210 CN**: 声明 TableGen def 记录 `A2_sub`。
- **L211 EN**: Declares TableGen def record `A2_subh_h16_hh`.
  **L211 CN**: 声明 TableGen def 记录 `A2_subh_h16_hh`。
- **L212 EN**: Declares TableGen def record `A2_subh_h16_hl`.
  **L212 CN**: 声明 TableGen def 记录 `A2_subh_h16_hl`。
- **L213 EN**: Declares TableGen def record `A2_subh_h16_lh`.
  **L213 CN**: 声明 TableGen def 记录 `A2_subh_h16_lh`。
- **L214 EN**: Declares TableGen def record `A2_subh_h16_ll`.
  **L214 CN**: 声明 TableGen def 记录 `A2_subh_h16_ll`。
- **L215 EN**: Declares TableGen def record `A2_subh_h16_sat_hh`.
  **L215 CN**: 声明 TableGen def 记录 `A2_subh_h16_sat_hh`。
- **L216 EN**: Declares TableGen def record `A2_subh_h16_sat_hl`.
  **L216 CN**: 声明 TableGen def 记录 `A2_subh_h16_sat_hl`。
- **L217 EN**: Declares TableGen def record `A2_subh_h16_sat_lh`.
  **L217 CN**: 声明 TableGen def 记录 `A2_subh_h16_sat_lh`。
- **L218 EN**: Declares TableGen def record `A2_subh_h16_sat_ll`.
  **L218 CN**: 声明 TableGen def 记录 `A2_subh_h16_sat_ll`。
- **L219 EN**: Declares TableGen def record `A2_subh_l16_hl`.
  **L219 CN**: 声明 TableGen def 记录 `A2_subh_l16_hl`。
- **L220 EN**: Declares TableGen def record `A2_subh_l16_ll`.
  **L220 CN**: 声明 TableGen def 记录 `A2_subh_l16_ll`。
- **L221 EN**: Declares TableGen def record `A2_subh_l16_sat_hl`.
  **L221 CN**: 声明 TableGen def 记录 `A2_subh_l16_sat_hl`。
- **L222 EN**: Declares TableGen def record `A2_subh_l16_sat_ll`.
  **L222 CN**: 声明 TableGen def 记录 `A2_subh_l16_sat_ll`。
- **L223 EN**: Declares TableGen def record `A2_subp`.
  **L223 CN**: 声明 TableGen def 记录 `A2_subp`。
- **L224 EN**: Declares TableGen def record `A2_subri`.
  **L224 CN**: 声明 TableGen def 记录 `A2_subri`。

### Lines 225-252

````tablegen
def A2_subsat : HexagonBuiltin<"int(int, int)">;
def A2_svaddh : HexagonBuiltin<"int(int, int)">;
def A2_svaddhs : HexagonBuiltin<"int(int, int)">;
def A2_svadduhs : HexagonBuiltin<"int(int, int)">;
def A2_svavgh : HexagonBuiltin<"int(int, int)">;
def A2_svavghs : HexagonBuiltin<"int(int, int)">;
def A2_svnavgh : HexagonBuiltin<"int(int, int)">;
def A2_svsubh : HexagonBuiltin<"int(int, int)">;
def A2_svsubhs : HexagonBuiltin<"int(int, int)">;
def A2_svsubuhs : HexagonBuiltin<"int(int, int)">;
def A2_swiz : HexagonBuiltin<"int(int)">;
def A2_sxtb : HexagonBuiltin<"int(int)">;
def A2_sxth : HexagonBuiltin<"int(int)">;
def A2_sxtw : HexagonBuiltin<"long long int(int)">;
def A2_tfr : HexagonBuiltin<"int(int)">;
def A2_tfrih : HexagonBuiltin<"int(int, unsigned _Constant int)">;
def A2_tfril : HexagonBuiltin<"int(int, unsigned _Constant int)">;
def A2_tfrp : HexagonBuiltin<"long long int(long long int)">;
def A2_tfrpi : HexagonBuiltin<"long long int(_Constant int)">;
def A2_tfrsi : HexagonBuiltin<"int(_Constant int)">;
def A2_vabsh : HexagonBuiltin<"long long int(long long int)">;
def A2_vabshsat : HexagonBuiltin<"long long int(long long int)">;
def A2_vabsw : HexagonBuiltin<"long long int(long long int)">;
def A2_vabswsat : HexagonBuiltin<"long long int(long long int)">;
def A2_vaddb_map : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vaddh : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vaddhs : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vaddub : HexagonBuiltin<"long long int(long long int, long long int)">;
````
- **L225 EN**: Declares TableGen def record `A2_subsat`.
  **L225 CN**: 声明 TableGen def 记录 `A2_subsat`。
- **L226 EN**: Declares TableGen def record `A2_svaddh`.
  **L226 CN**: 声明 TableGen def 记录 `A2_svaddh`。
- **L227 EN**: Declares TableGen def record `A2_svaddhs`.
  **L227 CN**: 声明 TableGen def 记录 `A2_svaddhs`。
- **L228 EN**: Declares TableGen def record `A2_svadduhs`.
  **L228 CN**: 声明 TableGen def 记录 `A2_svadduhs`。
- **L229 EN**: Declares TableGen def record `A2_svavgh`.
  **L229 CN**: 声明 TableGen def 记录 `A2_svavgh`。
- **L230 EN**: Declares TableGen def record `A2_svavghs`.
  **L230 CN**: 声明 TableGen def 记录 `A2_svavghs`。
- **L231 EN**: Declares TableGen def record `A2_svnavgh`.
  **L231 CN**: 声明 TableGen def 记录 `A2_svnavgh`。
- **L232 EN**: Declares TableGen def record `A2_svsubh`.
  **L232 CN**: 声明 TableGen def 记录 `A2_svsubh`。
- **L233 EN**: Declares TableGen def record `A2_svsubhs`.
  **L233 CN**: 声明 TableGen def 记录 `A2_svsubhs`。
- **L234 EN**: Declares TableGen def record `A2_svsubuhs`.
  **L234 CN**: 声明 TableGen def 记录 `A2_svsubuhs`。
- **L235 EN**: Declares TableGen def record `A2_swiz`.
  **L235 CN**: 声明 TableGen def 记录 `A2_swiz`。
- **L236 EN**: Declares TableGen def record `A2_sxtb`.
  **L236 CN**: 声明 TableGen def 记录 `A2_sxtb`。
- **L237 EN**: Declares TableGen def record `A2_sxth`.
  **L237 CN**: 声明 TableGen def 记录 `A2_sxth`。
- **L238 EN**: Declares TableGen def record `A2_sxtw`.
  **L238 CN**: 声明 TableGen def 记录 `A2_sxtw`。
- **L239 EN**: Declares TableGen def record `A2_tfr`.
  **L239 CN**: 声明 TableGen def 记录 `A2_tfr`。
- **L240 EN**: Declares TableGen def record `A2_tfrih`.
  **L240 CN**: 声明 TableGen def 记录 `A2_tfrih`。
- **L241 EN**: Declares TableGen def record `A2_tfril`.
  **L241 CN**: 声明 TableGen def 记录 `A2_tfril`。
- **L242 EN**: Declares TableGen def record `A2_tfrp`.
  **L242 CN**: 声明 TableGen def 记录 `A2_tfrp`。
- **L243 EN**: Declares TableGen def record `A2_tfrpi`.
  **L243 CN**: 声明 TableGen def 记录 `A2_tfrpi`。
- **L244 EN**: Declares TableGen def record `A2_tfrsi`.
  **L244 CN**: 声明 TableGen def 记录 `A2_tfrsi`。
- **L245 EN**: Declares TableGen def record `A2_vabsh`.
  **L245 CN**: 声明 TableGen def 记录 `A2_vabsh`。
- **L246 EN**: Declares TableGen def record `A2_vabshsat`.
  **L246 CN**: 声明 TableGen def 记录 `A2_vabshsat`。
- **L247 EN**: Declares TableGen def record `A2_vabsw`.
  **L247 CN**: 声明 TableGen def 记录 `A2_vabsw`。
- **L248 EN**: Declares TableGen def record `A2_vabswsat`.
  **L248 CN**: 声明 TableGen def 记录 `A2_vabswsat`。
- **L249 EN**: Declares TableGen def record `A2_vaddb_map`.
  **L249 CN**: 声明 TableGen def 记录 `A2_vaddb_map`。
- **L250 EN**: Declares TableGen def record `A2_vaddh`.
  **L250 CN**: 声明 TableGen def 记录 `A2_vaddh`。
- **L251 EN**: Declares TableGen def record `A2_vaddhs`.
  **L251 CN**: 声明 TableGen def 记录 `A2_vaddhs`。
- **L252 EN**: Declares TableGen def record `A2_vaddub`.
  **L252 CN**: 声明 TableGen def 记录 `A2_vaddub`。

### Lines 253-280

````tablegen
def A2_vaddubs : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vadduhs : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vaddw : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vaddws : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vavgh : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vavghcr : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vavghr : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vavgub : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vavgubr : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vavguh : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vavguhr : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vavguw : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vavguwr : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vavgw : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vavgwcr : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vavgwr : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vcmpbeq : HexagonBuiltin<"int(long long int, long long int)">;
def A2_vcmpbgtu : HexagonBuiltin<"int(long long int, long long int)">;
def A2_vcmpheq : HexagonBuiltin<"int(long long int, long long int)">;
def A2_vcmphgt : HexagonBuiltin<"int(long long int, long long int)">;
def A2_vcmphgtu : HexagonBuiltin<"int(long long int, long long int)">;
def A2_vcmpweq : HexagonBuiltin<"int(long long int, long long int)">;
def A2_vcmpwgt : HexagonBuiltin<"int(long long int, long long int)">;
def A2_vcmpwgtu : HexagonBuiltin<"int(long long int, long long int)">;
def A2_vconj : HexagonBuiltin<"long long int(long long int)">;
def A2_vmaxb : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vmaxh : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vmaxub : HexagonBuiltin<"long long int(long long int, long long int)">;
````
- **L253 EN**: Declares TableGen def record `A2_vaddubs`.
  **L253 CN**: 声明 TableGen def 记录 `A2_vaddubs`。
- **L254 EN**: Declares TableGen def record `A2_vadduhs`.
  **L254 CN**: 声明 TableGen def 记录 `A2_vadduhs`。
- **L255 EN**: Declares TableGen def record `A2_vaddw`.
  **L255 CN**: 声明 TableGen def 记录 `A2_vaddw`。
- **L256 EN**: Declares TableGen def record `A2_vaddws`.
  **L256 CN**: 声明 TableGen def 记录 `A2_vaddws`。
- **L257 EN**: Declares TableGen def record `A2_vavgh`.
  **L257 CN**: 声明 TableGen def 记录 `A2_vavgh`。
- **L258 EN**: Declares TableGen def record `A2_vavghcr`.
  **L258 CN**: 声明 TableGen def 记录 `A2_vavghcr`。
- **L259 EN**: Declares TableGen def record `A2_vavghr`.
  **L259 CN**: 声明 TableGen def 记录 `A2_vavghr`。
- **L260 EN**: Declares TableGen def record `A2_vavgub`.
  **L260 CN**: 声明 TableGen def 记录 `A2_vavgub`。
- **L261 EN**: Declares TableGen def record `A2_vavgubr`.
  **L261 CN**: 声明 TableGen def 记录 `A2_vavgubr`。
- **L262 EN**: Declares TableGen def record `A2_vavguh`.
  **L262 CN**: 声明 TableGen def 记录 `A2_vavguh`。
- **L263 EN**: Declares TableGen def record `A2_vavguhr`.
  **L263 CN**: 声明 TableGen def 记录 `A2_vavguhr`。
- **L264 EN**: Declares TableGen def record `A2_vavguw`.
  **L264 CN**: 声明 TableGen def 记录 `A2_vavguw`。
- **L265 EN**: Declares TableGen def record `A2_vavguwr`.
  **L265 CN**: 声明 TableGen def 记录 `A2_vavguwr`。
- **L266 EN**: Declares TableGen def record `A2_vavgw`.
  **L266 CN**: 声明 TableGen def 记录 `A2_vavgw`。
- **L267 EN**: Declares TableGen def record `A2_vavgwcr`.
  **L267 CN**: 声明 TableGen def 记录 `A2_vavgwcr`。
- **L268 EN**: Declares TableGen def record `A2_vavgwr`.
  **L268 CN**: 声明 TableGen def 记录 `A2_vavgwr`。
- **L269 EN**: Declares TableGen def record `A2_vcmpbeq`.
  **L269 CN**: 声明 TableGen def 记录 `A2_vcmpbeq`。
- **L270 EN**: Declares TableGen def record `A2_vcmpbgtu`.
  **L270 CN**: 声明 TableGen def 记录 `A2_vcmpbgtu`。
- **L271 EN**: Declares TableGen def record `A2_vcmpheq`.
  **L271 CN**: 声明 TableGen def 记录 `A2_vcmpheq`。
- **L272 EN**: Declares TableGen def record `A2_vcmphgt`.
  **L272 CN**: 声明 TableGen def 记录 `A2_vcmphgt`。
- **L273 EN**: Declares TableGen def record `A2_vcmphgtu`.
  **L273 CN**: 声明 TableGen def 记录 `A2_vcmphgtu`。
- **L274 EN**: Declares TableGen def record `A2_vcmpweq`.
  **L274 CN**: 声明 TableGen def 记录 `A2_vcmpweq`。
- **L275 EN**: Declares TableGen def record `A2_vcmpwgt`.
  **L275 CN**: 声明 TableGen def 记录 `A2_vcmpwgt`。
- **L276 EN**: Declares TableGen def record `A2_vcmpwgtu`.
  **L276 CN**: 声明 TableGen def 记录 `A2_vcmpwgtu`。
- **L277 EN**: Declares TableGen def record `A2_vconj`.
  **L277 CN**: 声明 TableGen def 记录 `A2_vconj`。
- **L278 EN**: Declares TableGen def record `A2_vmaxb`.
  **L278 CN**: 声明 TableGen def 记录 `A2_vmaxb`。
- **L279 EN**: Declares TableGen def record `A2_vmaxh`.
  **L279 CN**: 声明 TableGen def 记录 `A2_vmaxh`。
- **L280 EN**: Declares TableGen def record `A2_vmaxub`.
  **L280 CN**: 声明 TableGen def 记录 `A2_vmaxub`。

### Lines 281-308

````tablegen
def A2_vmaxuh : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vmaxuw : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vmaxw : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vminb : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vminh : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vminub : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vminuh : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vminuw : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vminw : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vnavgh : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vnavghcr : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vnavghr : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vnavgw : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vnavgwcr : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vnavgwr : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vraddub : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vraddub_acc : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
def A2_vrsadub : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vrsadub_acc : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
def A2_vsubb_map : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vsubh : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vsubhs : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vsubub : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vsububs : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vsubuhs : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vsubw : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_vsubws : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_xor : HexagonBuiltin<"int(int, int)">;
````
- **L281 EN**: Declares TableGen def record `A2_vmaxuh`.
  **L281 CN**: 声明 TableGen def 记录 `A2_vmaxuh`。
- **L282 EN**: Declares TableGen def record `A2_vmaxuw`.
  **L282 CN**: 声明 TableGen def 记录 `A2_vmaxuw`。
- **L283 EN**: Declares TableGen def record `A2_vmaxw`.
  **L283 CN**: 声明 TableGen def 记录 `A2_vmaxw`。
- **L284 EN**: Declares TableGen def record `A2_vminb`.
  **L284 CN**: 声明 TableGen def 记录 `A2_vminb`。
- **L285 EN**: Declares TableGen def record `A2_vminh`.
  **L285 CN**: 声明 TableGen def 记录 `A2_vminh`。
- **L286 EN**: Declares TableGen def record `A2_vminub`.
  **L286 CN**: 声明 TableGen def 记录 `A2_vminub`。
- **L287 EN**: Declares TableGen def record `A2_vminuh`.
  **L287 CN**: 声明 TableGen def 记录 `A2_vminuh`。
- **L288 EN**: Declares TableGen def record `A2_vminuw`.
  **L288 CN**: 声明 TableGen def 记录 `A2_vminuw`。
- **L289 EN**: Declares TableGen def record `A2_vminw`.
  **L289 CN**: 声明 TableGen def 记录 `A2_vminw`。
- **L290 EN**: Declares TableGen def record `A2_vnavgh`.
  **L290 CN**: 声明 TableGen def 记录 `A2_vnavgh`。
- **L291 EN**: Declares TableGen def record `A2_vnavghcr`.
  **L291 CN**: 声明 TableGen def 记录 `A2_vnavghcr`。
- **L292 EN**: Declares TableGen def record `A2_vnavghr`.
  **L292 CN**: 声明 TableGen def 记录 `A2_vnavghr`。
- **L293 EN**: Declares TableGen def record `A2_vnavgw`.
  **L293 CN**: 声明 TableGen def 记录 `A2_vnavgw`。
- **L294 EN**: Declares TableGen def record `A2_vnavgwcr`.
  **L294 CN**: 声明 TableGen def 记录 `A2_vnavgwcr`。
- **L295 EN**: Declares TableGen def record `A2_vnavgwr`.
  **L295 CN**: 声明 TableGen def 记录 `A2_vnavgwr`。
- **L296 EN**: Declares TableGen def record `A2_vraddub`.
  **L296 CN**: 声明 TableGen def 记录 `A2_vraddub`。
- **L297 EN**: Declares TableGen def record `A2_vraddub_acc`.
  **L297 CN**: 声明 TableGen def 记录 `A2_vraddub_acc`。
- **L298 EN**: Declares TableGen def record `A2_vrsadub`.
  **L298 CN**: 声明 TableGen def 记录 `A2_vrsadub`。
- **L299 EN**: Declares TableGen def record `A2_vrsadub_acc`.
  **L299 CN**: 声明 TableGen def 记录 `A2_vrsadub_acc`。
- **L300 EN**: Declares TableGen def record `A2_vsubb_map`.
  **L300 CN**: 声明 TableGen def 记录 `A2_vsubb_map`。
- **L301 EN**: Declares TableGen def record `A2_vsubh`.
  **L301 CN**: 声明 TableGen def 记录 `A2_vsubh`。
- **L302 EN**: Declares TableGen def record `A2_vsubhs`.
  **L302 CN**: 声明 TableGen def 记录 `A2_vsubhs`。
- **L303 EN**: Declares TableGen def record `A2_vsubub`.
  **L303 CN**: 声明 TableGen def 记录 `A2_vsubub`。
- **L304 EN**: Declares TableGen def record `A2_vsububs`.
  **L304 CN**: 声明 TableGen def 记录 `A2_vsububs`。
- **L305 EN**: Declares TableGen def record `A2_vsubuhs`.
  **L305 CN**: 声明 TableGen def 记录 `A2_vsubuhs`。
- **L306 EN**: Declares TableGen def record `A2_vsubw`.
  **L306 CN**: 声明 TableGen def 记录 `A2_vsubw`。
- **L307 EN**: Declares TableGen def record `A2_vsubws`.
  **L307 CN**: 声明 TableGen def 记录 `A2_vsubws`。
- **L308 EN**: Declares TableGen def record `A2_xor`.
  **L308 CN**: 声明 TableGen def 记录 `A2_xor`。

### Lines 309-336

````tablegen
def A2_xorp : HexagonBuiltin<"long long int(long long int, long long int)">;
def A2_zxtb : HexagonBuiltin<"int(int)">;
def A2_zxth : HexagonBuiltin<"int(int)">;
def A4_andn : HexagonBuiltin<"int(int, int)">;
def A4_andnp : HexagonBuiltin<"long long int(long long int, long long int)">;
def A4_bitsplit : HexagonBuiltin<"long long int(int, int)">;
def A4_bitspliti : HexagonBuiltin<"long long int(int, unsigned _Constant int)">;
def A4_boundscheck : HexagonBuiltin<"int(int, long long int)">;
def A4_cmpbeq : HexagonBuiltin<"int(int, int)">;
def A4_cmpbeqi : HexagonBuiltin<"int(int, unsigned _Constant int)">;
def A4_cmpbgt : HexagonBuiltin<"int(int, int)">;
def A4_cmpbgti : HexagonBuiltin<"int(int, _Constant int)">;
def A4_cmpbgtu : HexagonBuiltin<"int(int, int)">;
def A4_cmpbgtui : HexagonBuiltin<"int(int, unsigned _Constant int)">;
def A4_cmpheq : HexagonBuiltin<"int(int, int)">;
def A4_cmpheqi : HexagonBuiltin<"int(int, _Constant int)">;
def A4_cmphgt : HexagonBuiltin<"int(int, int)">;
def A4_cmphgti : HexagonBuiltin<"int(int, _Constant int)">;
def A4_cmphgtu : HexagonBuiltin<"int(int, int)">;
def A4_cmphgtui : HexagonBuiltin<"int(int, unsigned _Constant int)">;
def A4_combineir : HexagonBuiltin<"long long int(_Constant int, int)">;
def A4_combineri : HexagonBuiltin<"long long int(int, _Constant int)">;
def A4_cround_ri : HexagonBuiltin<"int(int, unsigned _Constant int)">;
def A4_cround_rr : HexagonBuiltin<"int(int, int)">;
def A4_modwrapu : HexagonBuiltin<"int(int, int)">;
def A4_orn : HexagonBuiltin<"int(int, int)">;
def A4_ornp : HexagonBuiltin<"long long int(long long int, long long int)">;
def A4_rcmpeq : HexagonBuiltin<"int(int, int)">;
````
- **L309 EN**: Declares TableGen def record `A2_xorp`.
  **L309 CN**: 声明 TableGen def 记录 `A2_xorp`。
- **L310 EN**: Declares TableGen def record `A2_zxtb`.
  **L310 CN**: 声明 TableGen def 记录 `A2_zxtb`。
- **L311 EN**: Declares TableGen def record `A2_zxth`.
  **L311 CN**: 声明 TableGen def 记录 `A2_zxth`。
- **L312 EN**: Declares TableGen def record `A4_andn`.
  **L312 CN**: 声明 TableGen def 记录 `A4_andn`。
- **L313 EN**: Declares TableGen def record `A4_andnp`.
  **L313 CN**: 声明 TableGen def 记录 `A4_andnp`。
- **L314 EN**: Declares TableGen def record `A4_bitsplit`.
  **L314 CN**: 声明 TableGen def 记录 `A4_bitsplit`。
- **L315 EN**: Declares TableGen def record `A4_bitspliti`.
  **L315 CN**: 声明 TableGen def 记录 `A4_bitspliti`。
- **L316 EN**: Declares TableGen def record `A4_boundscheck`.
  **L316 CN**: 声明 TableGen def 记录 `A4_boundscheck`。
- **L317 EN**: Declares TableGen def record `A4_cmpbeq`.
  **L317 CN**: 声明 TableGen def 记录 `A4_cmpbeq`。
- **L318 EN**: Declares TableGen def record `A4_cmpbeqi`.
  **L318 CN**: 声明 TableGen def 记录 `A4_cmpbeqi`。
- **L319 EN**: Declares TableGen def record `A4_cmpbgt`.
  **L319 CN**: 声明 TableGen def 记录 `A4_cmpbgt`。
- **L320 EN**: Declares TableGen def record `A4_cmpbgti`.
  **L320 CN**: 声明 TableGen def 记录 `A4_cmpbgti`。
- **L321 EN**: Declares TableGen def record `A4_cmpbgtu`.
  **L321 CN**: 声明 TableGen def 记录 `A4_cmpbgtu`。
- **L322 EN**: Declares TableGen def record `A4_cmpbgtui`.
  **L322 CN**: 声明 TableGen def 记录 `A4_cmpbgtui`。
- **L323 EN**: Declares TableGen def record `A4_cmpheq`.
  **L323 CN**: 声明 TableGen def 记录 `A4_cmpheq`。
- **L324 EN**: Declares TableGen def record `A4_cmpheqi`.
  **L324 CN**: 声明 TableGen def 记录 `A4_cmpheqi`。
- **L325 EN**: Declares TableGen def record `A4_cmphgt`.
  **L325 CN**: 声明 TableGen def 记录 `A4_cmphgt`。
- **L326 EN**: Declares TableGen def record `A4_cmphgti`.
  **L326 CN**: 声明 TableGen def 记录 `A4_cmphgti`。
- **L327 EN**: Declares TableGen def record `A4_cmphgtu`.
  **L327 CN**: 声明 TableGen def 记录 `A4_cmphgtu`。
- **L328 EN**: Declares TableGen def record `A4_cmphgtui`.
  **L328 CN**: 声明 TableGen def 记录 `A4_cmphgtui`。
- **L329 EN**: Declares TableGen def record `A4_combineir`.
  **L329 CN**: 声明 TableGen def 记录 `A4_combineir`。
- **L330 EN**: Declares TableGen def record `A4_combineri`.
  **L330 CN**: 声明 TableGen def 记录 `A4_combineri`。
- **L331 EN**: Declares TableGen def record `A4_cround_ri`.
  **L331 CN**: 声明 TableGen def 记录 `A4_cround_ri`。
- **L332 EN**: Declares TableGen def record `A4_cround_rr`.
  **L332 CN**: 声明 TableGen def 记录 `A4_cround_rr`。
- **L333 EN**: Declares TableGen def record `A4_modwrapu`.
  **L333 CN**: 声明 TableGen def 记录 `A4_modwrapu`。
- **L334 EN**: Declares TableGen def record `A4_orn`.
  **L334 CN**: 声明 TableGen def 记录 `A4_orn`。
- **L335 EN**: Declares TableGen def record `A4_ornp`.
  **L335 CN**: 声明 TableGen def 记录 `A4_ornp`。
- **L336 EN**: Declares TableGen def record `A4_rcmpeq`.
  **L336 CN**: 声明 TableGen def 记录 `A4_rcmpeq`。

### Lines 337-364

````tablegen
def A4_rcmpeqi : HexagonBuiltin<"int(int, _Constant int)">;
def A4_rcmpneq : HexagonBuiltin<"int(int, int)">;
def A4_rcmpneqi : HexagonBuiltin<"int(int, _Constant int)">;
def A4_round_ri : HexagonBuiltin<"int(int, unsigned _Constant int)">;
def A4_round_ri_sat : HexagonBuiltin<"int(int, unsigned _Constant int)">;
def A4_round_rr : HexagonBuiltin<"int(int, int)">;
def A4_round_rr_sat : HexagonBuiltin<"int(int, int)">;
def A4_tlbmatch : HexagonBuiltin<"int(long long int, int)">;
def A4_vcmpbeq_any : HexagonBuiltin<"int(long long int, long long int)">;
def A4_vcmpbeqi : HexagonBuiltin<"int(long long int, unsigned _Constant int)">;
def A4_vcmpbgt : HexagonBuiltin<"int(long long int, long long int)">;
def A4_vcmpbgti : HexagonBuiltin<"int(long long int, _Constant int)">;
def A4_vcmpbgtui : HexagonBuiltin<"int(long long int, unsigned _Constant int)">;
def A4_vcmpheqi : HexagonBuiltin<"int(long long int, _Constant int)">;
def A4_vcmphgti : HexagonBuiltin<"int(long long int, _Constant int)">;
def A4_vcmphgtui : HexagonBuiltin<"int(long long int, unsigned _Constant int)">;
def A4_vcmpweqi : HexagonBuiltin<"int(long long int, _Constant int)">;
def A4_vcmpwgti : HexagonBuiltin<"int(long long int, _Constant int)">;
def A4_vcmpwgtui : HexagonBuiltin<"int(long long int, unsigned _Constant int)">;
def A4_vrmaxh : HexagonBuiltin<"long long int(long long int, long long int, int)">;
def A4_vrmaxuh : HexagonBuiltin<"long long int(long long int, long long int, int)">;
def A4_vrmaxuw : HexagonBuiltin<"long long int(long long int, long long int, int)">;
def A4_vrmaxw : HexagonBuiltin<"long long int(long long int, long long int, int)">;
def A4_vrminh : HexagonBuiltin<"long long int(long long int, long long int, int)">;
def A4_vrminuh : HexagonBuiltin<"long long int(long long int, long long int, int)">;
def A4_vrminuw : HexagonBuiltin<"long long int(long long int, long long int, int)">;
def A4_vrminw : HexagonBuiltin<"long long int(long long int, long long int, int)">;
def A5_vaddhubs : HexagonBuiltin<"int(long long int, long long int)">;
````
- **L337 EN**: Declares TableGen def record `A4_rcmpeqi`.
  **L337 CN**: 声明 TableGen def 记录 `A4_rcmpeqi`。
- **L338 EN**: Declares TableGen def record `A4_rcmpneq`.
  **L338 CN**: 声明 TableGen def 记录 `A4_rcmpneq`。
- **L339 EN**: Declares TableGen def record `A4_rcmpneqi`.
  **L339 CN**: 声明 TableGen def 记录 `A4_rcmpneqi`。
- **L340 EN**: Declares TableGen def record `A4_round_ri`.
  **L340 CN**: 声明 TableGen def 记录 `A4_round_ri`。
- **L341 EN**: Declares TableGen def record `A4_round_ri_sat`.
  **L341 CN**: 声明 TableGen def 记录 `A4_round_ri_sat`。
- **L342 EN**: Declares TableGen def record `A4_round_rr`.
  **L342 CN**: 声明 TableGen def 记录 `A4_round_rr`。
- **L343 EN**: Declares TableGen def record `A4_round_rr_sat`.
  **L343 CN**: 声明 TableGen def 记录 `A4_round_rr_sat`。
- **L344 EN**: Declares TableGen def record `A4_tlbmatch`.
  **L344 CN**: 声明 TableGen def 记录 `A4_tlbmatch`。
- **L345 EN**: Declares TableGen def record `A4_vcmpbeq_any`.
  **L345 CN**: 声明 TableGen def 记录 `A4_vcmpbeq_any`。
- **L346 EN**: Declares TableGen def record `A4_vcmpbeqi`.
  **L346 CN**: 声明 TableGen def 记录 `A4_vcmpbeqi`。
- **L347 EN**: Declares TableGen def record `A4_vcmpbgt`.
  **L347 CN**: 声明 TableGen def 记录 `A4_vcmpbgt`。
- **L348 EN**: Declares TableGen def record `A4_vcmpbgti`.
  **L348 CN**: 声明 TableGen def 记录 `A4_vcmpbgti`。
- **L349 EN**: Declares TableGen def record `A4_vcmpbgtui`.
  **L349 CN**: 声明 TableGen def 记录 `A4_vcmpbgtui`。
- **L350 EN**: Declares TableGen def record `A4_vcmpheqi`.
  **L350 CN**: 声明 TableGen def 记录 `A4_vcmpheqi`。
- **L351 EN**: Declares TableGen def record `A4_vcmphgti`.
  **L351 CN**: 声明 TableGen def 记录 `A4_vcmphgti`。
- **L352 EN**: Declares TableGen def record `A4_vcmphgtui`.
  **L352 CN**: 声明 TableGen def 记录 `A4_vcmphgtui`。
- **L353 EN**: Declares TableGen def record `A4_vcmpweqi`.
  **L353 CN**: 声明 TableGen def 记录 `A4_vcmpweqi`。
- **L354 EN**: Declares TableGen def record `A4_vcmpwgti`.
  **L354 CN**: 声明 TableGen def 记录 `A4_vcmpwgti`。
- **L355 EN**: Declares TableGen def record `A4_vcmpwgtui`.
  **L355 CN**: 声明 TableGen def 记录 `A4_vcmpwgtui`。
- **L356 EN**: Declares TableGen def record `A4_vrmaxh`.
  **L356 CN**: 声明 TableGen def 记录 `A4_vrmaxh`。
- **L357 EN**: Declares TableGen def record `A4_vrmaxuh`.
  **L357 CN**: 声明 TableGen def 记录 `A4_vrmaxuh`。
- **L358 EN**: Declares TableGen def record `A4_vrmaxuw`.
  **L358 CN**: 声明 TableGen def 记录 `A4_vrmaxuw`。
- **L359 EN**: Declares TableGen def record `A4_vrmaxw`.
  **L359 CN**: 声明 TableGen def 记录 `A4_vrmaxw`。
- **L360 EN**: Declares TableGen def record `A4_vrminh`.
  **L360 CN**: 声明 TableGen def 记录 `A4_vrminh`。
- **L361 EN**: Declares TableGen def record `A4_vrminuh`.
  **L361 CN**: 声明 TableGen def 记录 `A4_vrminuh`。
- **L362 EN**: Declares TableGen def record `A4_vrminuw`.
  **L362 CN**: 声明 TableGen def 记录 `A4_vrminuw`。
- **L363 EN**: Declares TableGen def record `A4_vrminw`.
  **L363 CN**: 声明 TableGen def 记录 `A4_vrminw`。
- **L364 EN**: Declares TableGen def record `A5_vaddhubs`.
  **L364 CN**: 声明 TableGen def 记录 `A5_vaddhubs`。

### Lines 365-392

````tablegen
def C2_all8 : HexagonBuiltin<"int(int)">;
def C2_and : HexagonBuiltin<"int(int, int)">;
def C2_andn : HexagonBuiltin<"int(int, int)">;
def C2_any8 : HexagonBuiltin<"int(int)">;
def C2_bitsclr : HexagonBuiltin<"int(int, int)">;
def C2_bitsclri : HexagonBuiltin<"int(int, unsigned _Constant int)">;
def C2_bitsset : HexagonBuiltin<"int(int, int)">;
def C2_cmpeq : HexagonBuiltin<"int(int, int)">;
def C2_cmpeqi : HexagonBuiltin<"int(int, _Constant int)">;
def C2_cmpeqp : HexagonBuiltin<"int(long long int, long long int)">;
def C2_cmpgei : HexagonBuiltin<"int(int, _Constant int)">;
def C2_cmpgeui : HexagonBuiltin<"int(int, unsigned _Constant int)">;
def C2_cmpgt : HexagonBuiltin<"int(int, int)">;
def C2_cmpgti : HexagonBuiltin<"int(int, _Constant int)">;
def C2_cmpgtp : HexagonBuiltin<"int(long long int, long long int)">;
def C2_cmpgtu : HexagonBuiltin<"int(int, int)">;
def C2_cmpgtui : HexagonBuiltin<"int(int, unsigned _Constant int)">;
def C2_cmpgtup : HexagonBuiltin<"int(long long int, long long int)">;
def C2_cmplt : HexagonBuiltin<"int(int, int)">;
def C2_cmpltu : HexagonBuiltin<"int(int, int)">;
def C2_mask : HexagonBuiltin<"long long int(int)">;
def C2_mux : HexagonBuiltin<"int(int, int, int)">;
def C2_muxii : HexagonBuiltin<"int(int, _Constant int, _Constant int)">;
def C2_muxir : HexagonBuiltin<"int(int, int, _Constant int)">;
def C2_muxri : HexagonBuiltin<"int(int, _Constant int, int)">;
def C2_not : HexagonBuiltin<"int(int)">;
def C2_or : HexagonBuiltin<"int(int, int)">;
def C2_orn : HexagonBuiltin<"int(int, int)">;
````
- **L365 EN**: Declares TableGen def record `C2_all8`.
  **L365 CN**: 声明 TableGen def 记录 `C2_all8`。
- **L366 EN**: Declares TableGen def record `C2_and`.
  **L366 CN**: 声明 TableGen def 记录 `C2_and`。
- **L367 EN**: Declares TableGen def record `C2_andn`.
  **L367 CN**: 声明 TableGen def 记录 `C2_andn`。
- **L368 EN**: Declares TableGen def record `C2_any8`.
  **L368 CN**: 声明 TableGen def 记录 `C2_any8`。
- **L369 EN**: Declares TableGen def record `C2_bitsclr`.
  **L369 CN**: 声明 TableGen def 记录 `C2_bitsclr`。
- **L370 EN**: Declares TableGen def record `C2_bitsclri`.
  **L370 CN**: 声明 TableGen def 记录 `C2_bitsclri`。
- **L371 EN**: Declares TableGen def record `C2_bitsset`.
  **L371 CN**: 声明 TableGen def 记录 `C2_bitsset`。
- **L372 EN**: Declares TableGen def record `C2_cmpeq`.
  **L372 CN**: 声明 TableGen def 记录 `C2_cmpeq`。
- **L373 EN**: Declares TableGen def record `C2_cmpeqi`.
  **L373 CN**: 声明 TableGen def 记录 `C2_cmpeqi`。
- **L374 EN**: Declares TableGen def record `C2_cmpeqp`.
  **L374 CN**: 声明 TableGen def 记录 `C2_cmpeqp`。
- **L375 EN**: Declares TableGen def record `C2_cmpgei`.
  **L375 CN**: 声明 TableGen def 记录 `C2_cmpgei`。
- **L376 EN**: Declares TableGen def record `C2_cmpgeui`.
  **L376 CN**: 声明 TableGen def 记录 `C2_cmpgeui`。
- **L377 EN**: Declares TableGen def record `C2_cmpgt`.
  **L377 CN**: 声明 TableGen def 记录 `C2_cmpgt`。
- **L378 EN**: Declares TableGen def record `C2_cmpgti`.
  **L378 CN**: 声明 TableGen def 记录 `C2_cmpgti`。
- **L379 EN**: Declares TableGen def record `C2_cmpgtp`.
  **L379 CN**: 声明 TableGen def 记录 `C2_cmpgtp`。
- **L380 EN**: Declares TableGen def record `C2_cmpgtu`.
  **L380 CN**: 声明 TableGen def 记录 `C2_cmpgtu`。
- **L381 EN**: Declares TableGen def record `C2_cmpgtui`.
  **L381 CN**: 声明 TableGen def 记录 `C2_cmpgtui`。
- **L382 EN**: Declares TableGen def record `C2_cmpgtup`.
  **L382 CN**: 声明 TableGen def 记录 `C2_cmpgtup`。
- **L383 EN**: Declares TableGen def record `C2_cmplt`.
  **L383 CN**: 声明 TableGen def 记录 `C2_cmplt`。
- **L384 EN**: Declares TableGen def record `C2_cmpltu`.
  **L384 CN**: 声明 TableGen def 记录 `C2_cmpltu`。
- **L385 EN**: Declares TableGen def record `C2_mask`.
  **L385 CN**: 声明 TableGen def 记录 `C2_mask`。
- **L386 EN**: Declares TableGen def record `C2_mux`.
  **L386 CN**: 声明 TableGen def 记录 `C2_mux`。
- **L387 EN**: Declares TableGen def record `C2_muxii`.
  **L387 CN**: 声明 TableGen def 记录 `C2_muxii`。
- **L388 EN**: Declares TableGen def record `C2_muxir`.
  **L388 CN**: 声明 TableGen def 记录 `C2_muxir`。
- **L389 EN**: Declares TableGen def record `C2_muxri`.
  **L389 CN**: 声明 TableGen def 记录 `C2_muxri`。
- **L390 EN**: Declares TableGen def record `C2_not`.
  **L390 CN**: 声明 TableGen def 记录 `C2_not`。
- **L391 EN**: Declares TableGen def record `C2_or`.
  **L391 CN**: 声明 TableGen def 记录 `C2_or`。
- **L392 EN**: Declares TableGen def record `C2_orn`.
  **L392 CN**: 声明 TableGen def 记录 `C2_orn`。

### Lines 393-420

````tablegen
def C2_pxfer_map : HexagonBuiltin<"int(int)">;
def C2_tfrpr : HexagonBuiltin<"int(int)">;
def C2_tfrrp : HexagonBuiltin<"int(int)">;
def C2_vitpack : HexagonBuiltin<"int(int, int)">;
def C2_vmux : HexagonBuiltin<"long long int(int, long long int, long long int)">;
def C2_xor : HexagonBuiltin<"int(int, int)">;
def C4_and_and : HexagonBuiltin<"int(int, int, int)">;
def C4_and_andn : HexagonBuiltin<"int(int, int, int)">;
def C4_and_or : HexagonBuiltin<"int(int, int, int)">;
def C4_and_orn : HexagonBuiltin<"int(int, int, int)">;
def C4_cmplte : HexagonBuiltin<"int(int, int)">;
def C4_cmpltei : HexagonBuiltin<"int(int, _Constant int)">;
def C4_cmplteu : HexagonBuiltin<"int(int, int)">;
def C4_cmplteui : HexagonBuiltin<"int(int, unsigned _Constant int)">;
def C4_cmpneq : HexagonBuiltin<"int(int, int)">;
def C4_cmpneqi : HexagonBuiltin<"int(int, _Constant int)">;
def C4_fastcorner9 : HexagonBuiltin<"int(int, int)">;
def C4_fastcorner9_not : HexagonBuiltin<"int(int, int)">;
def C4_nbitsclr : HexagonBuiltin<"int(int, int)">;
def C4_nbitsclri : HexagonBuiltin<"int(int, unsigned _Constant int)">;
def C4_nbitsset : HexagonBuiltin<"int(int, int)">;
def C4_or_and : HexagonBuiltin<"int(int, int, int)">;
def C4_or_andn : HexagonBuiltin<"int(int, int, int)">;
def C4_or_or : HexagonBuiltin<"int(int, int, int)">;
def C4_or_orn : HexagonBuiltin<"int(int, int, int)">;
def F2_conv_d2df : HexagonBuiltin<"double(long long int)">;
def F2_conv_d2sf : HexagonBuiltin<"float(long long int)">;
def F2_conv_df2d : HexagonBuiltin<"long long int(double)">;
````
- **L393 EN**: Declares TableGen def record `C2_pxfer_map`.
  **L393 CN**: 声明 TableGen def 记录 `C2_pxfer_map`。
- **L394 EN**: Declares TableGen def record `C2_tfrpr`.
  **L394 CN**: 声明 TableGen def 记录 `C2_tfrpr`。
- **L395 EN**: Declares TableGen def record `C2_tfrrp`.
  **L395 CN**: 声明 TableGen def 记录 `C2_tfrrp`。
- **L396 EN**: Declares TableGen def record `C2_vitpack`.
  **L396 CN**: 声明 TableGen def 记录 `C2_vitpack`。
- **L397 EN**: Declares TableGen def record `C2_vmux`.
  **L397 CN**: 声明 TableGen def 记录 `C2_vmux`。
- **L398 EN**: Declares TableGen def record `C2_xor`.
  **L398 CN**: 声明 TableGen def 记录 `C2_xor`。
- **L399 EN**: Declares TableGen def record `C4_and_and`.
  **L399 CN**: 声明 TableGen def 记录 `C4_and_and`。
- **L400 EN**: Declares TableGen def record `C4_and_andn`.
  **L400 CN**: 声明 TableGen def 记录 `C4_and_andn`。
- **L401 EN**: Declares TableGen def record `C4_and_or`.
  **L401 CN**: 声明 TableGen def 记录 `C4_and_or`。
- **L402 EN**: Declares TableGen def record `C4_and_orn`.
  **L402 CN**: 声明 TableGen def 记录 `C4_and_orn`。
- **L403 EN**: Declares TableGen def record `C4_cmplte`.
  **L403 CN**: 声明 TableGen def 记录 `C4_cmplte`。
- **L404 EN**: Declares TableGen def record `C4_cmpltei`.
  **L404 CN**: 声明 TableGen def 记录 `C4_cmpltei`。
- **L405 EN**: Declares TableGen def record `C4_cmplteu`.
  **L405 CN**: 声明 TableGen def 记录 `C4_cmplteu`。
- **L406 EN**: Declares TableGen def record `C4_cmplteui`.
  **L406 CN**: 声明 TableGen def 记录 `C4_cmplteui`。
- **L407 EN**: Declares TableGen def record `C4_cmpneq`.
  **L407 CN**: 声明 TableGen def 记录 `C4_cmpneq`。
- **L408 EN**: Declares TableGen def record `C4_cmpneqi`.
  **L408 CN**: 声明 TableGen def 记录 `C4_cmpneqi`。
- **L409 EN**: Declares TableGen def record `C4_fastcorner9`.
  **L409 CN**: 声明 TableGen def 记录 `C4_fastcorner9`。
- **L410 EN**: Declares TableGen def record `C4_fastcorner9_not`.
  **L410 CN**: 声明 TableGen def 记录 `C4_fastcorner9_not`。
- **L411 EN**: Declares TableGen def record `C4_nbitsclr`.
  **L411 CN**: 声明 TableGen def 记录 `C4_nbitsclr`。
- **L412 EN**: Declares TableGen def record `C4_nbitsclri`.
  **L412 CN**: 声明 TableGen def 记录 `C4_nbitsclri`。
- **L413 EN**: Declares TableGen def record `C4_nbitsset`.
  **L413 CN**: 声明 TableGen def 记录 `C4_nbitsset`。
- **L414 EN**: Declares TableGen def record `C4_or_and`.
  **L414 CN**: 声明 TableGen def 记录 `C4_or_and`。
- **L415 EN**: Declares TableGen def record `C4_or_andn`.
  **L415 CN**: 声明 TableGen def 记录 `C4_or_andn`。
- **L416 EN**: Declares TableGen def record `C4_or_or`.
  **L416 CN**: 声明 TableGen def 记录 `C4_or_or`。
- **L417 EN**: Declares TableGen def record `C4_or_orn`.
  **L417 CN**: 声明 TableGen def 记录 `C4_or_orn`。
- **L418 EN**: Declares TableGen def record `F2_conv_d2df`.
  **L418 CN**: 声明 TableGen def 记录 `F2_conv_d2df`。
- **L419 EN**: Declares TableGen def record `F2_conv_d2sf`.
  **L419 CN**: 声明 TableGen def 记录 `F2_conv_d2sf`。
- **L420 EN**: Declares TableGen def record `F2_conv_df2d`.
  **L420 CN**: 声明 TableGen def 记录 `F2_conv_df2d`。

### Lines 421-448

````tablegen
def F2_conv_df2d_chop : HexagonBuiltin<"long long int(double)">;
def F2_conv_df2sf : HexagonBuiltin<"float(double)">;
def F2_conv_df2ud : HexagonBuiltin<"long long int(double)">;
def F2_conv_df2ud_chop : HexagonBuiltin<"long long int(double)">;
def F2_conv_df2uw : HexagonBuiltin<"int(double)">;
def F2_conv_df2uw_chop : HexagonBuiltin<"int(double)">;
def F2_conv_df2w : HexagonBuiltin<"int(double)">;
def F2_conv_df2w_chop : HexagonBuiltin<"int(double)">;
def F2_conv_sf2d : HexagonBuiltin<"long long int(float)">;
def F2_conv_sf2d_chop : HexagonBuiltin<"long long int(float)">;
def F2_conv_sf2df : HexagonBuiltin<"double(float)">;
def F2_conv_sf2ud : HexagonBuiltin<"long long int(float)">;
def F2_conv_sf2ud_chop : HexagonBuiltin<"long long int(float)">;
def F2_conv_sf2uw : HexagonBuiltin<"int(float)">;
def F2_conv_sf2uw_chop : HexagonBuiltin<"int(float)">;
def F2_conv_sf2w : HexagonBuiltin<"int(float)">;
def F2_conv_sf2w_chop : HexagonBuiltin<"int(float)">;
def F2_conv_ud2df : HexagonBuiltin<"double(long long int)">;
def F2_conv_ud2sf : HexagonBuiltin<"float(long long int)">;
def F2_conv_uw2df : HexagonBuiltin<"double(int)">;
def F2_conv_uw2sf : HexagonBuiltin<"float(int)">;
def F2_conv_w2df : HexagonBuiltin<"double(int)">;
def F2_conv_w2sf : HexagonBuiltin<"float(int)">;
def F2_dfclass : HexagonBuiltin<"int(double, unsigned _Constant int)">;
def F2_dfcmpeq : HexagonBuiltin<"int(double, double)">;
def F2_dfcmpge : HexagonBuiltin<"int(double, double)">;
def F2_dfcmpgt : HexagonBuiltin<"int(double, double)">;
def F2_dfcmpuo : HexagonBuiltin<"int(double, double)">;
````
- **L421 EN**: Declares TableGen def record `F2_conv_df2d_chop`.
  **L421 CN**: 声明 TableGen def 记录 `F2_conv_df2d_chop`。
- **L422 EN**: Declares TableGen def record `F2_conv_df2sf`.
  **L422 CN**: 声明 TableGen def 记录 `F2_conv_df2sf`。
- **L423 EN**: Declares TableGen def record `F2_conv_df2ud`.
  **L423 CN**: 声明 TableGen def 记录 `F2_conv_df2ud`。
- **L424 EN**: Declares TableGen def record `F2_conv_df2ud_chop`.
  **L424 CN**: 声明 TableGen def 记录 `F2_conv_df2ud_chop`。
- **L425 EN**: Declares TableGen def record `F2_conv_df2uw`.
  **L425 CN**: 声明 TableGen def 记录 `F2_conv_df2uw`。
- **L426 EN**: Declares TableGen def record `F2_conv_df2uw_chop`.
  **L426 CN**: 声明 TableGen def 记录 `F2_conv_df2uw_chop`。
- **L427 EN**: Declares TableGen def record `F2_conv_df2w`.
  **L427 CN**: 声明 TableGen def 记录 `F2_conv_df2w`。
- **L428 EN**: Declares TableGen def record `F2_conv_df2w_chop`.
  **L428 CN**: 声明 TableGen def 记录 `F2_conv_df2w_chop`。
- **L429 EN**: Declares TableGen def record `F2_conv_sf2d`.
  **L429 CN**: 声明 TableGen def 记录 `F2_conv_sf2d`。
- **L430 EN**: Declares TableGen def record `F2_conv_sf2d_chop`.
  **L430 CN**: 声明 TableGen def 记录 `F2_conv_sf2d_chop`。
- **L431 EN**: Declares TableGen def record `F2_conv_sf2df`.
  **L431 CN**: 声明 TableGen def 记录 `F2_conv_sf2df`。
- **L432 EN**: Declares TableGen def record `F2_conv_sf2ud`.
  **L432 CN**: 声明 TableGen def 记录 `F2_conv_sf2ud`。
- **L433 EN**: Declares TableGen def record `F2_conv_sf2ud_chop`.
  **L433 CN**: 声明 TableGen def 记录 `F2_conv_sf2ud_chop`。
- **L434 EN**: Declares TableGen def record `F2_conv_sf2uw`.
  **L434 CN**: 声明 TableGen def 记录 `F2_conv_sf2uw`。
- **L435 EN**: Declares TableGen def record `F2_conv_sf2uw_chop`.
  **L435 CN**: 声明 TableGen def 记录 `F2_conv_sf2uw_chop`。
- **L436 EN**: Declares TableGen def record `F2_conv_sf2w`.
  **L436 CN**: 声明 TableGen def 记录 `F2_conv_sf2w`。
- **L437 EN**: Declares TableGen def record `F2_conv_sf2w_chop`.
  **L437 CN**: 声明 TableGen def 记录 `F2_conv_sf2w_chop`。
- **L438 EN**: Declares TableGen def record `F2_conv_ud2df`.
  **L438 CN**: 声明 TableGen def 记录 `F2_conv_ud2df`。
- **L439 EN**: Declares TableGen def record `F2_conv_ud2sf`.
  **L439 CN**: 声明 TableGen def 记录 `F2_conv_ud2sf`。
- **L440 EN**: Declares TableGen def record `F2_conv_uw2df`.
  **L440 CN**: 声明 TableGen def 记录 `F2_conv_uw2df`。
- **L441 EN**: Declares TableGen def record `F2_conv_uw2sf`.
  **L441 CN**: 声明 TableGen def 记录 `F2_conv_uw2sf`。
- **L442 EN**: Declares TableGen def record `F2_conv_w2df`.
  **L442 CN**: 声明 TableGen def 记录 `F2_conv_w2df`。
- **L443 EN**: Declares TableGen def record `F2_conv_w2sf`.
  **L443 CN**: 声明 TableGen def 记录 `F2_conv_w2sf`。
- **L444 EN**: Declares TableGen def record `F2_dfclass`.
  **L444 CN**: 声明 TableGen def 记录 `F2_dfclass`。
- **L445 EN**: Declares TableGen def record `F2_dfcmpeq`.
  **L445 CN**: 声明 TableGen def 记录 `F2_dfcmpeq`。
- **L446 EN**: Declares TableGen def record `F2_dfcmpge`.
  **L446 CN**: 声明 TableGen def 记录 `F2_dfcmpge`。
- **L447 EN**: Declares TableGen def record `F2_dfcmpgt`.
  **L447 CN**: 声明 TableGen def 记录 `F2_dfcmpgt`。
- **L448 EN**: Declares TableGen def record `F2_dfcmpuo`.
  **L448 CN**: 声明 TableGen def 记录 `F2_dfcmpuo`。

### Lines 449-476

````tablegen
def F2_dfimm_n : HexagonBuiltin<"double(unsigned _Constant int)">;
def F2_dfimm_p : HexagonBuiltin<"double(unsigned _Constant int)">;
def F2_sfadd : HexagonBuiltin<"float(float, float)">;
def F2_sfclass : HexagonBuiltin<"int(float, unsigned _Constant int)">;
def F2_sfcmpeq : HexagonBuiltin<"int(float, float)">;
def F2_sfcmpge : HexagonBuiltin<"int(float, float)">;
def F2_sfcmpgt : HexagonBuiltin<"int(float, float)">;
def F2_sfcmpuo : HexagonBuiltin<"int(float, float)">;
def F2_sffixupd : HexagonBuiltin<"float(float, float)">;
def F2_sffixupn : HexagonBuiltin<"float(float, float)">;
def F2_sffixupr : HexagonBuiltin<"float(float)">;
def F2_sffma : HexagonBuiltin<"float(float, float, float)">;
def F2_sffma_lib : HexagonBuiltin<"float(float, float, float)">;
def F2_sffma_sc : HexagonBuiltin<"float(float, float, float, int)">;
def F2_sffms : HexagonBuiltin<"float(float, float, float)">;
def F2_sffms_lib : HexagonBuiltin<"float(float, float, float)">;
def F2_sfimm_n : HexagonBuiltin<"float(unsigned _Constant int)">;
def F2_sfimm_p : HexagonBuiltin<"float(unsigned _Constant int)">;
def F2_sfmax : HexagonBuiltin<"float(float, float)">;
def F2_sfmin : HexagonBuiltin<"float(float, float)">;
def F2_sfmpy : HexagonBuiltin<"float(float, float)">;
def F2_sfsub : HexagonBuiltin<"float(float, float)">;
def M2_acci : HexagonBuiltin<"int(int, int, int)">;
def M2_accii : HexagonBuiltin<"int(int, int, _Constant int)">;
def M2_cmaci_s0 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_cmacr_s0 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_cmacs_s0 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_cmacs_s1 : HexagonBuiltin<"long long int(long long int, int, int)">;
````
- **L449 EN**: Declares TableGen def record `F2_dfimm_n`.
  **L449 CN**: 声明 TableGen def 记录 `F2_dfimm_n`。
- **L450 EN**: Declares TableGen def record `F2_dfimm_p`.
  **L450 CN**: 声明 TableGen def 记录 `F2_dfimm_p`。
- **L451 EN**: Declares TableGen def record `F2_sfadd`.
  **L451 CN**: 声明 TableGen def 记录 `F2_sfadd`。
- **L452 EN**: Declares TableGen def record `F2_sfclass`.
  **L452 CN**: 声明 TableGen def 记录 `F2_sfclass`。
- **L453 EN**: Declares TableGen def record `F2_sfcmpeq`.
  **L453 CN**: 声明 TableGen def 记录 `F2_sfcmpeq`。
- **L454 EN**: Declares TableGen def record `F2_sfcmpge`.
  **L454 CN**: 声明 TableGen def 记录 `F2_sfcmpge`。
- **L455 EN**: Declares TableGen def record `F2_sfcmpgt`.
  **L455 CN**: 声明 TableGen def 记录 `F2_sfcmpgt`。
- **L456 EN**: Declares TableGen def record `F2_sfcmpuo`.
  **L456 CN**: 声明 TableGen def 记录 `F2_sfcmpuo`。
- **L457 EN**: Declares TableGen def record `F2_sffixupd`.
  **L457 CN**: 声明 TableGen def 记录 `F2_sffixupd`。
- **L458 EN**: Declares TableGen def record `F2_sffixupn`.
  **L458 CN**: 声明 TableGen def 记录 `F2_sffixupn`。
- **L459 EN**: Declares TableGen def record `F2_sffixupr`.
  **L459 CN**: 声明 TableGen def 记录 `F2_sffixupr`。
- **L460 EN**: Declares TableGen def record `F2_sffma`.
  **L460 CN**: 声明 TableGen def 记录 `F2_sffma`。
- **L461 EN**: Declares TableGen def record `F2_sffma_lib`.
  **L461 CN**: 声明 TableGen def 记录 `F2_sffma_lib`。
- **L462 EN**: Declares TableGen def record `F2_sffma_sc`.
  **L462 CN**: 声明 TableGen def 记录 `F2_sffma_sc`。
- **L463 EN**: Declares TableGen def record `F2_sffms`.
  **L463 CN**: 声明 TableGen def 记录 `F2_sffms`。
- **L464 EN**: Declares TableGen def record `F2_sffms_lib`.
  **L464 CN**: 声明 TableGen def 记录 `F2_sffms_lib`。
- **L465 EN**: Declares TableGen def record `F2_sfimm_n`.
  **L465 CN**: 声明 TableGen def 记录 `F2_sfimm_n`。
- **L466 EN**: Declares TableGen def record `F2_sfimm_p`.
  **L466 CN**: 声明 TableGen def 记录 `F2_sfimm_p`。
- **L467 EN**: Declares TableGen def record `F2_sfmax`.
  **L467 CN**: 声明 TableGen def 记录 `F2_sfmax`。
- **L468 EN**: Declares TableGen def record `F2_sfmin`.
  **L468 CN**: 声明 TableGen def 记录 `F2_sfmin`。
- **L469 EN**: Declares TableGen def record `F2_sfmpy`.
  **L469 CN**: 声明 TableGen def 记录 `F2_sfmpy`。
- **L470 EN**: Declares TableGen def record `F2_sfsub`.
  **L470 CN**: 声明 TableGen def 记录 `F2_sfsub`。
- **L471 EN**: Declares TableGen def record `M2_acci`.
  **L471 CN**: 声明 TableGen def 记录 `M2_acci`。
- **L472 EN**: Declares TableGen def record `M2_accii`.
  **L472 CN**: 声明 TableGen def 记录 `M2_accii`。
- **L473 EN**: Declares TableGen def record `M2_cmaci_s0`.
  **L473 CN**: 声明 TableGen def 记录 `M2_cmaci_s0`。
- **L474 EN**: Declares TableGen def record `M2_cmacr_s0`.
  **L474 CN**: 声明 TableGen def 记录 `M2_cmacr_s0`。
- **L475 EN**: Declares TableGen def record `M2_cmacs_s0`.
  **L475 CN**: 声明 TableGen def 记录 `M2_cmacs_s0`。
- **L476 EN**: Declares TableGen def record `M2_cmacs_s1`.
  **L476 CN**: 声明 TableGen def 记录 `M2_cmacs_s1`。

### Lines 477-504

````tablegen
def M2_cmacsc_s0 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_cmacsc_s1 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_cmpyi_s0 : HexagonBuiltin<"long long int(int, int)">;
def M2_cmpyr_s0 : HexagonBuiltin<"long long int(int, int)">;
def M2_cmpyrs_s0 : HexagonBuiltin<"int(int, int)">;
def M2_cmpyrs_s1 : HexagonBuiltin<"int(int, int)">;
def M2_cmpyrsc_s0 : HexagonBuiltin<"int(int, int)">;
def M2_cmpyrsc_s1 : HexagonBuiltin<"int(int, int)">;
def M2_cmpys_s0 : HexagonBuiltin<"long long int(int, int)">;
def M2_cmpys_s1 : HexagonBuiltin<"long long int(int, int)">;
def M2_cmpysc_s0 : HexagonBuiltin<"long long int(int, int)">;
def M2_cmpysc_s1 : HexagonBuiltin<"long long int(int, int)">;
def M2_cnacs_s0 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_cnacs_s1 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_cnacsc_s0 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_cnacsc_s1 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_dpmpyss_acc_s0 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_dpmpyss_nac_s0 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_dpmpyss_rnd_s0 : HexagonBuiltin<"int(int, int)">;
def M2_dpmpyss_s0 : HexagonBuiltin<"long long int(int, int)">;
def M2_dpmpyuu_acc_s0 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_dpmpyuu_nac_s0 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_dpmpyuu_s0 : HexagonBuiltin<"unsigned long long int(int, int)">;
def M2_hmmpyh_rs1 : HexagonBuiltin<"int(int, int)">;
def M2_hmmpyh_s1 : HexagonBuiltin<"int(int, int)">;
def M2_hmmpyl_rs1 : HexagonBuiltin<"int(int, int)">;
def M2_hmmpyl_s1 : HexagonBuiltin<"int(int, int)">;
def M2_maci : HexagonBuiltin<"int(int, int, int)">;
````
- **L477 EN**: Declares TableGen def record `M2_cmacsc_s0`.
  **L477 CN**: 声明 TableGen def 记录 `M2_cmacsc_s0`。
- **L478 EN**: Declares TableGen def record `M2_cmacsc_s1`.
  **L478 CN**: 声明 TableGen def 记录 `M2_cmacsc_s1`。
- **L479 EN**: Declares TableGen def record `M2_cmpyi_s0`.
  **L479 CN**: 声明 TableGen def 记录 `M2_cmpyi_s0`。
- **L480 EN**: Declares TableGen def record `M2_cmpyr_s0`.
  **L480 CN**: 声明 TableGen def 记录 `M2_cmpyr_s0`。
- **L481 EN**: Declares TableGen def record `M2_cmpyrs_s0`.
  **L481 CN**: 声明 TableGen def 记录 `M2_cmpyrs_s0`。
- **L482 EN**: Declares TableGen def record `M2_cmpyrs_s1`.
  **L482 CN**: 声明 TableGen def 记录 `M2_cmpyrs_s1`。
- **L483 EN**: Declares TableGen def record `M2_cmpyrsc_s0`.
  **L483 CN**: 声明 TableGen def 记录 `M2_cmpyrsc_s0`。
- **L484 EN**: Declares TableGen def record `M2_cmpyrsc_s1`.
  **L484 CN**: 声明 TableGen def 记录 `M2_cmpyrsc_s1`。
- **L485 EN**: Declares TableGen def record `M2_cmpys_s0`.
  **L485 CN**: 声明 TableGen def 记录 `M2_cmpys_s0`。
- **L486 EN**: Declares TableGen def record `M2_cmpys_s1`.
  **L486 CN**: 声明 TableGen def 记录 `M2_cmpys_s1`。
- **L487 EN**: Declares TableGen def record `M2_cmpysc_s0`.
  **L487 CN**: 声明 TableGen def 记录 `M2_cmpysc_s0`。
- **L488 EN**: Declares TableGen def record `M2_cmpysc_s1`.
  **L488 CN**: 声明 TableGen def 记录 `M2_cmpysc_s1`。
- **L489 EN**: Declares TableGen def record `M2_cnacs_s0`.
  **L489 CN**: 声明 TableGen def 记录 `M2_cnacs_s0`。
- **L490 EN**: Declares TableGen def record `M2_cnacs_s1`.
  **L490 CN**: 声明 TableGen def 记录 `M2_cnacs_s1`。
- **L491 EN**: Declares TableGen def record `M2_cnacsc_s0`.
  **L491 CN**: 声明 TableGen def 记录 `M2_cnacsc_s0`。
- **L492 EN**: Declares TableGen def record `M2_cnacsc_s1`.
  **L492 CN**: 声明 TableGen def 记录 `M2_cnacsc_s1`。
- **L493 EN**: Declares TableGen def record `M2_dpmpyss_acc_s0`.
  **L493 CN**: 声明 TableGen def 记录 `M2_dpmpyss_acc_s0`。
- **L494 EN**: Declares TableGen def record `M2_dpmpyss_nac_s0`.
  **L494 CN**: 声明 TableGen def 记录 `M2_dpmpyss_nac_s0`。
- **L495 EN**: Declares TableGen def record `M2_dpmpyss_rnd_s0`.
  **L495 CN**: 声明 TableGen def 记录 `M2_dpmpyss_rnd_s0`。
- **L496 EN**: Declares TableGen def record `M2_dpmpyss_s0`.
  **L496 CN**: 声明 TableGen def 记录 `M2_dpmpyss_s0`。
- **L497 EN**: Declares TableGen def record `M2_dpmpyuu_acc_s0`.
  **L497 CN**: 声明 TableGen def 记录 `M2_dpmpyuu_acc_s0`。
- **L498 EN**: Declares TableGen def record `M2_dpmpyuu_nac_s0`.
  **L498 CN**: 声明 TableGen def 记录 `M2_dpmpyuu_nac_s0`。
- **L499 EN**: Declares TableGen def record `M2_dpmpyuu_s0`.
  **L499 CN**: 声明 TableGen def 记录 `M2_dpmpyuu_s0`。
- **L500 EN**: Declares TableGen def record `M2_hmmpyh_rs1`.
  **L500 CN**: 声明 TableGen def 记录 `M2_hmmpyh_rs1`。
- **L501 EN**: Declares TableGen def record `M2_hmmpyh_s1`.
  **L501 CN**: 声明 TableGen def 记录 `M2_hmmpyh_s1`。
- **L502 EN**: Declares TableGen def record `M2_hmmpyl_rs1`.
  **L502 CN**: 声明 TableGen def 记录 `M2_hmmpyl_rs1`。
- **L503 EN**: Declares TableGen def record `M2_hmmpyl_s1`.
  **L503 CN**: 声明 TableGen def 记录 `M2_hmmpyl_s1`。
- **L504 EN**: Declares TableGen def record `M2_maci`.
  **L504 CN**: 声明 TableGen def 记录 `M2_maci`。

### Lines 505-532

````tablegen
def M2_macsin : HexagonBuiltin<"int(int, int, unsigned _Constant int)">;
def M2_macsip : HexagonBuiltin<"int(int, int, unsigned _Constant int)">;
def M2_mmachs_rs0 : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
def M2_mmachs_rs1 : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
def M2_mmachs_s0 : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
def M2_mmachs_s1 : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
def M2_mmacls_rs0 : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
def M2_mmacls_rs1 : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
def M2_mmacls_s0 : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
def M2_mmacls_s1 : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
def M2_mmacuhs_rs0 : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
def M2_mmacuhs_rs1 : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
def M2_mmacuhs_s0 : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
def M2_mmacuhs_s1 : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
def M2_mmaculs_rs0 : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
def M2_mmaculs_rs1 : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
def M2_mmaculs_s0 : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
def M2_mmaculs_s1 : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
def M2_mmpyh_rs0 : HexagonBuiltin<"long long int(long long int, long long int)">;
def M2_mmpyh_rs1 : HexagonBuiltin<"long long int(long long int, long long int)">;
def M2_mmpyh_s0 : HexagonBuiltin<"long long int(long long int, long long int)">;
def M2_mmpyh_s1 : HexagonBuiltin<"long long int(long long int, long long int)">;
def M2_mmpyl_rs0 : HexagonBuiltin<"long long int(long long int, long long int)">;
def M2_mmpyl_rs1 : HexagonBuiltin<"long long int(long long int, long long int)">;
def M2_mmpyl_s0 : HexagonBuiltin<"long long int(long long int, long long int)">;
def M2_mmpyl_s1 : HexagonBuiltin<"long long int(long long int, long long int)">;
def M2_mmpyuh_rs0 : HexagonBuiltin<"long long int(long long int, long long int)">;
def M2_mmpyuh_rs1 : HexagonBuiltin<"long long int(long long int, long long int)">;
````
- **L505 EN**: Declares TableGen def record `M2_macsin`.
  **L505 CN**: 声明 TableGen def 记录 `M2_macsin`。
- **L506 EN**: Declares TableGen def record `M2_macsip`.
  **L506 CN**: 声明 TableGen def 记录 `M2_macsip`。
- **L507 EN**: Declares TableGen def record `M2_mmachs_rs0`.
  **L507 CN**: 声明 TableGen def 记录 `M2_mmachs_rs0`。
- **L508 EN**: Declares TableGen def record `M2_mmachs_rs1`.
  **L508 CN**: 声明 TableGen def 记录 `M2_mmachs_rs1`。
- **L509 EN**: Declares TableGen def record `M2_mmachs_s0`.
  **L509 CN**: 声明 TableGen def 记录 `M2_mmachs_s0`。
- **L510 EN**: Declares TableGen def record `M2_mmachs_s1`.
  **L510 CN**: 声明 TableGen def 记录 `M2_mmachs_s1`。
- **L511 EN**: Declares TableGen def record `M2_mmacls_rs0`.
  **L511 CN**: 声明 TableGen def 记录 `M2_mmacls_rs0`。
- **L512 EN**: Declares TableGen def record `M2_mmacls_rs1`.
  **L512 CN**: 声明 TableGen def 记录 `M2_mmacls_rs1`。
- **L513 EN**: Declares TableGen def record `M2_mmacls_s0`.
  **L513 CN**: 声明 TableGen def 记录 `M2_mmacls_s0`。
- **L514 EN**: Declares TableGen def record `M2_mmacls_s1`.
  **L514 CN**: 声明 TableGen def 记录 `M2_mmacls_s1`。
- **L515 EN**: Declares TableGen def record `M2_mmacuhs_rs0`.
  **L515 CN**: 声明 TableGen def 记录 `M2_mmacuhs_rs0`。
- **L516 EN**: Declares TableGen def record `M2_mmacuhs_rs1`.
  **L516 CN**: 声明 TableGen def 记录 `M2_mmacuhs_rs1`。
- **L517 EN**: Declares TableGen def record `M2_mmacuhs_s0`.
  **L517 CN**: 声明 TableGen def 记录 `M2_mmacuhs_s0`。
- **L518 EN**: Declares TableGen def record `M2_mmacuhs_s1`.
  **L518 CN**: 声明 TableGen def 记录 `M2_mmacuhs_s1`。
- **L519 EN**: Declares TableGen def record `M2_mmaculs_rs0`.
  **L519 CN**: 声明 TableGen def 记录 `M2_mmaculs_rs0`。
- **L520 EN**: Declares TableGen def record `M2_mmaculs_rs1`.
  **L520 CN**: 声明 TableGen def 记录 `M2_mmaculs_rs1`。
- **L521 EN**: Declares TableGen def record `M2_mmaculs_s0`.
  **L521 CN**: 声明 TableGen def 记录 `M2_mmaculs_s0`。
- **L522 EN**: Declares TableGen def record `M2_mmaculs_s1`.
  **L522 CN**: 声明 TableGen def 记录 `M2_mmaculs_s1`。
- **L523 EN**: Declares TableGen def record `M2_mmpyh_rs0`.
  **L523 CN**: 声明 TableGen def 记录 `M2_mmpyh_rs0`。
- **L524 EN**: Declares TableGen def record `M2_mmpyh_rs1`.
  **L524 CN**: 声明 TableGen def 记录 `M2_mmpyh_rs1`。
- **L525 EN**: Declares TableGen def record `M2_mmpyh_s0`.
  **L525 CN**: 声明 TableGen def 记录 `M2_mmpyh_s0`。
- **L526 EN**: Declares TableGen def record `M2_mmpyh_s1`.
  **L526 CN**: 声明 TableGen def 记录 `M2_mmpyh_s1`。
- **L527 EN**: Declares TableGen def record `M2_mmpyl_rs0`.
  **L527 CN**: 声明 TableGen def 记录 `M2_mmpyl_rs0`。
- **L528 EN**: Declares TableGen def record `M2_mmpyl_rs1`.
  **L528 CN**: 声明 TableGen def 记录 `M2_mmpyl_rs1`。
- **L529 EN**: Declares TableGen def record `M2_mmpyl_s0`.
  **L529 CN**: 声明 TableGen def 记录 `M2_mmpyl_s0`。
- **L530 EN**: Declares TableGen def record `M2_mmpyl_s1`.
  **L530 CN**: 声明 TableGen def 记录 `M2_mmpyl_s1`。
- **L531 EN**: Declares TableGen def record `M2_mmpyuh_rs0`.
  **L531 CN**: 声明 TableGen def 记录 `M2_mmpyuh_rs0`。
- **L532 EN**: Declares TableGen def record `M2_mmpyuh_rs1`.
  **L532 CN**: 声明 TableGen def 记录 `M2_mmpyuh_rs1`。

### Lines 533-560

````tablegen
def M2_mmpyuh_s0 : HexagonBuiltin<"long long int(long long int, long long int)">;
def M2_mmpyuh_s1 : HexagonBuiltin<"long long int(long long int, long long int)">;
def M2_mmpyul_rs0 : HexagonBuiltin<"long long int(long long int, long long int)">;
def M2_mmpyul_rs1 : HexagonBuiltin<"long long int(long long int, long long int)">;
def M2_mmpyul_s0 : HexagonBuiltin<"long long int(long long int, long long int)">;
def M2_mmpyul_s1 : HexagonBuiltin<"long long int(long long int, long long int)">;
def M2_mpy_acc_hh_s0 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpy_acc_hh_s1 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpy_acc_hl_s0 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpy_acc_hl_s1 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpy_acc_lh_s0 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpy_acc_lh_s1 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpy_acc_ll_s0 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpy_acc_ll_s1 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpy_acc_sat_hh_s0 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpy_acc_sat_hh_s1 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpy_acc_sat_hl_s0 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpy_acc_sat_hl_s1 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpy_acc_sat_lh_s0 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpy_acc_sat_lh_s1 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpy_acc_sat_ll_s0 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpy_acc_sat_ll_s1 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpy_hh_s0 : HexagonBuiltin<"int(int, int)">;
def M2_mpy_hh_s1 : HexagonBuiltin<"int(int, int)">;
def M2_mpy_hl_s0 : HexagonBuiltin<"int(int, int)">;
def M2_mpy_hl_s1 : HexagonBuiltin<"int(int, int)">;
def M2_mpy_lh_s0 : HexagonBuiltin<"int(int, int)">;
def M2_mpy_lh_s1 : HexagonBuiltin<"int(int, int)">;
````
- **L533 EN**: Declares TableGen def record `M2_mmpyuh_s0`.
  **L533 CN**: 声明 TableGen def 记录 `M2_mmpyuh_s0`。
- **L534 EN**: Declares TableGen def record `M2_mmpyuh_s1`.
  **L534 CN**: 声明 TableGen def 记录 `M2_mmpyuh_s1`。
- **L535 EN**: Declares TableGen def record `M2_mmpyul_rs0`.
  **L535 CN**: 声明 TableGen def 记录 `M2_mmpyul_rs0`。
- **L536 EN**: Declares TableGen def record `M2_mmpyul_rs1`.
  **L536 CN**: 声明 TableGen def 记录 `M2_mmpyul_rs1`。
- **L537 EN**: Declares TableGen def record `M2_mmpyul_s0`.
  **L537 CN**: 声明 TableGen def 记录 `M2_mmpyul_s0`。
- **L538 EN**: Declares TableGen def record `M2_mmpyul_s1`.
  **L538 CN**: 声明 TableGen def 记录 `M2_mmpyul_s1`。
- **L539 EN**: Declares TableGen def record `M2_mpy_acc_hh_s0`.
  **L539 CN**: 声明 TableGen def 记录 `M2_mpy_acc_hh_s0`。
- **L540 EN**: Declares TableGen def record `M2_mpy_acc_hh_s1`.
  **L540 CN**: 声明 TableGen def 记录 `M2_mpy_acc_hh_s1`。
- **L541 EN**: Declares TableGen def record `M2_mpy_acc_hl_s0`.
  **L541 CN**: 声明 TableGen def 记录 `M2_mpy_acc_hl_s0`。
- **L542 EN**: Declares TableGen def record `M2_mpy_acc_hl_s1`.
  **L542 CN**: 声明 TableGen def 记录 `M2_mpy_acc_hl_s1`。
- **L543 EN**: Declares TableGen def record `M2_mpy_acc_lh_s0`.
  **L543 CN**: 声明 TableGen def 记录 `M2_mpy_acc_lh_s0`。
- **L544 EN**: Declares TableGen def record `M2_mpy_acc_lh_s1`.
  **L544 CN**: 声明 TableGen def 记录 `M2_mpy_acc_lh_s1`。
- **L545 EN**: Declares TableGen def record `M2_mpy_acc_ll_s0`.
  **L545 CN**: 声明 TableGen def 记录 `M2_mpy_acc_ll_s0`。
- **L546 EN**: Declares TableGen def record `M2_mpy_acc_ll_s1`.
  **L546 CN**: 声明 TableGen def 记录 `M2_mpy_acc_ll_s1`。
- **L547 EN**: Declares TableGen def record `M2_mpy_acc_sat_hh_s0`.
  **L547 CN**: 声明 TableGen def 记录 `M2_mpy_acc_sat_hh_s0`。
- **L548 EN**: Declares TableGen def record `M2_mpy_acc_sat_hh_s1`.
  **L548 CN**: 声明 TableGen def 记录 `M2_mpy_acc_sat_hh_s1`。
- **L549 EN**: Declares TableGen def record `M2_mpy_acc_sat_hl_s0`.
  **L549 CN**: 声明 TableGen def 记录 `M2_mpy_acc_sat_hl_s0`。
- **L550 EN**: Declares TableGen def record `M2_mpy_acc_sat_hl_s1`.
  **L550 CN**: 声明 TableGen def 记录 `M2_mpy_acc_sat_hl_s1`。
- **L551 EN**: Declares TableGen def record `M2_mpy_acc_sat_lh_s0`.
  **L551 CN**: 声明 TableGen def 记录 `M2_mpy_acc_sat_lh_s0`。
- **L552 EN**: Declares TableGen def record `M2_mpy_acc_sat_lh_s1`.
  **L552 CN**: 声明 TableGen def 记录 `M2_mpy_acc_sat_lh_s1`。
- **L553 EN**: Declares TableGen def record `M2_mpy_acc_sat_ll_s0`.
  **L553 CN**: 声明 TableGen def 记录 `M2_mpy_acc_sat_ll_s0`。
- **L554 EN**: Declares TableGen def record `M2_mpy_acc_sat_ll_s1`.
  **L554 CN**: 声明 TableGen def 记录 `M2_mpy_acc_sat_ll_s1`。
- **L555 EN**: Declares TableGen def record `M2_mpy_hh_s0`.
  **L555 CN**: 声明 TableGen def 记录 `M2_mpy_hh_s0`。
- **L556 EN**: Declares TableGen def record `M2_mpy_hh_s1`.
  **L556 CN**: 声明 TableGen def 记录 `M2_mpy_hh_s1`。
- **L557 EN**: Declares TableGen def record `M2_mpy_hl_s0`.
  **L557 CN**: 声明 TableGen def 记录 `M2_mpy_hl_s0`。
- **L558 EN**: Declares TableGen def record `M2_mpy_hl_s1`.
  **L558 CN**: 声明 TableGen def 记录 `M2_mpy_hl_s1`。
- **L559 EN**: Declares TableGen def record `M2_mpy_lh_s0`.
  **L559 CN**: 声明 TableGen def 记录 `M2_mpy_lh_s0`。
- **L560 EN**: Declares TableGen def record `M2_mpy_lh_s1`.
  **L560 CN**: 声明 TableGen def 记录 `M2_mpy_lh_s1`。

### Lines 561-588

````tablegen
def M2_mpy_ll_s0 : HexagonBuiltin<"int(int, int)">;
def M2_mpy_ll_s1 : HexagonBuiltin<"int(int, int)">;
def M2_mpy_nac_hh_s0 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpy_nac_hh_s1 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpy_nac_hl_s0 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpy_nac_hl_s1 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpy_nac_lh_s0 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpy_nac_lh_s1 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpy_nac_ll_s0 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpy_nac_ll_s1 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpy_nac_sat_hh_s0 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpy_nac_sat_hh_s1 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpy_nac_sat_hl_s0 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpy_nac_sat_hl_s1 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpy_nac_sat_lh_s0 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpy_nac_sat_lh_s1 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpy_nac_sat_ll_s0 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpy_nac_sat_ll_s1 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpy_rnd_hh_s0 : HexagonBuiltin<"int(int, int)">;
def M2_mpy_rnd_hh_s1 : HexagonBuiltin<"int(int, int)">;
def M2_mpy_rnd_hl_s0 : HexagonBuiltin<"int(int, int)">;
def M2_mpy_rnd_hl_s1 : HexagonBuiltin<"int(int, int)">;
def M2_mpy_rnd_lh_s0 : HexagonBuiltin<"int(int, int)">;
def M2_mpy_rnd_lh_s1 : HexagonBuiltin<"int(int, int)">;
def M2_mpy_rnd_ll_s0 : HexagonBuiltin<"int(int, int)">;
def M2_mpy_rnd_ll_s1 : HexagonBuiltin<"int(int, int)">;
def M2_mpy_sat_hh_s0 : HexagonBuiltin<"int(int, int)">;
def M2_mpy_sat_hh_s1 : HexagonBuiltin<"int(int, int)">;
````
- **L561 EN**: Declares TableGen def record `M2_mpy_ll_s0`.
  **L561 CN**: 声明 TableGen def 记录 `M2_mpy_ll_s0`。
- **L562 EN**: Declares TableGen def record `M2_mpy_ll_s1`.
  **L562 CN**: 声明 TableGen def 记录 `M2_mpy_ll_s1`。
- **L563 EN**: Declares TableGen def record `M2_mpy_nac_hh_s0`.
  **L563 CN**: 声明 TableGen def 记录 `M2_mpy_nac_hh_s0`。
- **L564 EN**: Declares TableGen def record `M2_mpy_nac_hh_s1`.
  **L564 CN**: 声明 TableGen def 记录 `M2_mpy_nac_hh_s1`。
- **L565 EN**: Declares TableGen def record `M2_mpy_nac_hl_s0`.
  **L565 CN**: 声明 TableGen def 记录 `M2_mpy_nac_hl_s0`。
- **L566 EN**: Declares TableGen def record `M2_mpy_nac_hl_s1`.
  **L566 CN**: 声明 TableGen def 记录 `M2_mpy_nac_hl_s1`。
- **L567 EN**: Declares TableGen def record `M2_mpy_nac_lh_s0`.
  **L567 CN**: 声明 TableGen def 记录 `M2_mpy_nac_lh_s0`。
- **L568 EN**: Declares TableGen def record `M2_mpy_nac_lh_s1`.
  **L568 CN**: 声明 TableGen def 记录 `M2_mpy_nac_lh_s1`。
- **L569 EN**: Declares TableGen def record `M2_mpy_nac_ll_s0`.
  **L569 CN**: 声明 TableGen def 记录 `M2_mpy_nac_ll_s0`。
- **L570 EN**: Declares TableGen def record `M2_mpy_nac_ll_s1`.
  **L570 CN**: 声明 TableGen def 记录 `M2_mpy_nac_ll_s1`。
- **L571 EN**: Declares TableGen def record `M2_mpy_nac_sat_hh_s0`.
  **L571 CN**: 声明 TableGen def 记录 `M2_mpy_nac_sat_hh_s0`。
- **L572 EN**: Declares TableGen def record `M2_mpy_nac_sat_hh_s1`.
  **L572 CN**: 声明 TableGen def 记录 `M2_mpy_nac_sat_hh_s1`。
- **L573 EN**: Declares TableGen def record `M2_mpy_nac_sat_hl_s0`.
  **L573 CN**: 声明 TableGen def 记录 `M2_mpy_nac_sat_hl_s0`。
- **L574 EN**: Declares TableGen def record `M2_mpy_nac_sat_hl_s1`.
  **L574 CN**: 声明 TableGen def 记录 `M2_mpy_nac_sat_hl_s1`。
- **L575 EN**: Declares TableGen def record `M2_mpy_nac_sat_lh_s0`.
  **L575 CN**: 声明 TableGen def 记录 `M2_mpy_nac_sat_lh_s0`。
- **L576 EN**: Declares TableGen def record `M2_mpy_nac_sat_lh_s1`.
  **L576 CN**: 声明 TableGen def 记录 `M2_mpy_nac_sat_lh_s1`。
- **L577 EN**: Declares TableGen def record `M2_mpy_nac_sat_ll_s0`.
  **L577 CN**: 声明 TableGen def 记录 `M2_mpy_nac_sat_ll_s0`。
- **L578 EN**: Declares TableGen def record `M2_mpy_nac_sat_ll_s1`.
  **L578 CN**: 声明 TableGen def 记录 `M2_mpy_nac_sat_ll_s1`。
- **L579 EN**: Declares TableGen def record `M2_mpy_rnd_hh_s0`.
  **L579 CN**: 声明 TableGen def 记录 `M2_mpy_rnd_hh_s0`。
- **L580 EN**: Declares TableGen def record `M2_mpy_rnd_hh_s1`.
  **L580 CN**: 声明 TableGen def 记录 `M2_mpy_rnd_hh_s1`。
- **L581 EN**: Declares TableGen def record `M2_mpy_rnd_hl_s0`.
  **L581 CN**: 声明 TableGen def 记录 `M2_mpy_rnd_hl_s0`。
- **L582 EN**: Declares TableGen def record `M2_mpy_rnd_hl_s1`.
  **L582 CN**: 声明 TableGen def 记录 `M2_mpy_rnd_hl_s1`。
- **L583 EN**: Declares TableGen def record `M2_mpy_rnd_lh_s0`.
  **L583 CN**: 声明 TableGen def 记录 `M2_mpy_rnd_lh_s0`。
- **L584 EN**: Declares TableGen def record `M2_mpy_rnd_lh_s1`.
  **L584 CN**: 声明 TableGen def 记录 `M2_mpy_rnd_lh_s1`。
- **L585 EN**: Declares TableGen def record `M2_mpy_rnd_ll_s0`.
  **L585 CN**: 声明 TableGen def 记录 `M2_mpy_rnd_ll_s0`。
- **L586 EN**: Declares TableGen def record `M2_mpy_rnd_ll_s1`.
  **L586 CN**: 声明 TableGen def 记录 `M2_mpy_rnd_ll_s1`。
- **L587 EN**: Declares TableGen def record `M2_mpy_sat_hh_s0`.
  **L587 CN**: 声明 TableGen def 记录 `M2_mpy_sat_hh_s0`。
- **L588 EN**: Declares TableGen def record `M2_mpy_sat_hh_s1`.
  **L588 CN**: 声明 TableGen def 记录 `M2_mpy_sat_hh_s1`。

### Lines 589-616

````tablegen
def M2_mpy_sat_hl_s0 : HexagonBuiltin<"int(int, int)">;
def M2_mpy_sat_hl_s1 : HexagonBuiltin<"int(int, int)">;
def M2_mpy_sat_lh_s0 : HexagonBuiltin<"int(int, int)">;
def M2_mpy_sat_lh_s1 : HexagonBuiltin<"int(int, int)">;
def M2_mpy_sat_ll_s0 : HexagonBuiltin<"int(int, int)">;
def M2_mpy_sat_ll_s1 : HexagonBuiltin<"int(int, int)">;
def M2_mpy_sat_rnd_hh_s0 : HexagonBuiltin<"int(int, int)">;
def M2_mpy_sat_rnd_hh_s1 : HexagonBuiltin<"int(int, int)">;
def M2_mpy_sat_rnd_hl_s0 : HexagonBuiltin<"int(int, int)">;
def M2_mpy_sat_rnd_hl_s1 : HexagonBuiltin<"int(int, int)">;
def M2_mpy_sat_rnd_lh_s0 : HexagonBuiltin<"int(int, int)">;
def M2_mpy_sat_rnd_lh_s1 : HexagonBuiltin<"int(int, int)">;
def M2_mpy_sat_rnd_ll_s0 : HexagonBuiltin<"int(int, int)">;
def M2_mpy_sat_rnd_ll_s1 : HexagonBuiltin<"int(int, int)">;
def M2_mpy_up : HexagonBuiltin<"int(int, int)">;
def M2_mpy_up_s1 : HexagonBuiltin<"int(int, int)">;
def M2_mpy_up_s1_sat : HexagonBuiltin<"int(int, int)">;
def M2_mpyd_acc_hh_s0 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_mpyd_acc_hh_s1 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_mpyd_acc_hl_s0 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_mpyd_acc_hl_s1 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_mpyd_acc_lh_s0 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_mpyd_acc_lh_s1 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_mpyd_acc_ll_s0 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_mpyd_acc_ll_s1 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_mpyd_hh_s0 : HexagonBuiltin<"long long int(int, int)">;
def M2_mpyd_hh_s1 : HexagonBuiltin<"long long int(int, int)">;
def M2_mpyd_hl_s0 : HexagonBuiltin<"long long int(int, int)">;
````
- **L589 EN**: Declares TableGen def record `M2_mpy_sat_hl_s0`.
  **L589 CN**: 声明 TableGen def 记录 `M2_mpy_sat_hl_s0`。
- **L590 EN**: Declares TableGen def record `M2_mpy_sat_hl_s1`.
  **L590 CN**: 声明 TableGen def 记录 `M2_mpy_sat_hl_s1`。
- **L591 EN**: Declares TableGen def record `M2_mpy_sat_lh_s0`.
  **L591 CN**: 声明 TableGen def 记录 `M2_mpy_sat_lh_s0`。
- **L592 EN**: Declares TableGen def record `M2_mpy_sat_lh_s1`.
  **L592 CN**: 声明 TableGen def 记录 `M2_mpy_sat_lh_s1`。
- **L593 EN**: Declares TableGen def record `M2_mpy_sat_ll_s0`.
  **L593 CN**: 声明 TableGen def 记录 `M2_mpy_sat_ll_s0`。
- **L594 EN**: Declares TableGen def record `M2_mpy_sat_ll_s1`.
  **L594 CN**: 声明 TableGen def 记录 `M2_mpy_sat_ll_s1`。
- **L595 EN**: Declares TableGen def record `M2_mpy_sat_rnd_hh_s0`.
  **L595 CN**: 声明 TableGen def 记录 `M2_mpy_sat_rnd_hh_s0`。
- **L596 EN**: Declares TableGen def record `M2_mpy_sat_rnd_hh_s1`.
  **L596 CN**: 声明 TableGen def 记录 `M2_mpy_sat_rnd_hh_s1`。
- **L597 EN**: Declares TableGen def record `M2_mpy_sat_rnd_hl_s0`.
  **L597 CN**: 声明 TableGen def 记录 `M2_mpy_sat_rnd_hl_s0`。
- **L598 EN**: Declares TableGen def record `M2_mpy_sat_rnd_hl_s1`.
  **L598 CN**: 声明 TableGen def 记录 `M2_mpy_sat_rnd_hl_s1`。
- **L599 EN**: Declares TableGen def record `M2_mpy_sat_rnd_lh_s0`.
  **L599 CN**: 声明 TableGen def 记录 `M2_mpy_sat_rnd_lh_s0`。
- **L600 EN**: Declares TableGen def record `M2_mpy_sat_rnd_lh_s1`.
  **L600 CN**: 声明 TableGen def 记录 `M2_mpy_sat_rnd_lh_s1`。
- **L601 EN**: Declares TableGen def record `M2_mpy_sat_rnd_ll_s0`.
  **L601 CN**: 声明 TableGen def 记录 `M2_mpy_sat_rnd_ll_s0`。
- **L602 EN**: Declares TableGen def record `M2_mpy_sat_rnd_ll_s1`.
  **L602 CN**: 声明 TableGen def 记录 `M2_mpy_sat_rnd_ll_s1`。
- **L603 EN**: Declares TableGen def record `M2_mpy_up`.
  **L603 CN**: 声明 TableGen def 记录 `M2_mpy_up`。
- **L604 EN**: Declares TableGen def record `M2_mpy_up_s1`.
  **L604 CN**: 声明 TableGen def 记录 `M2_mpy_up_s1`。
- **L605 EN**: Declares TableGen def record `M2_mpy_up_s1_sat`.
  **L605 CN**: 声明 TableGen def 记录 `M2_mpy_up_s1_sat`。
- **L606 EN**: Declares TableGen def record `M2_mpyd_acc_hh_s0`.
  **L606 CN**: 声明 TableGen def 记录 `M2_mpyd_acc_hh_s0`。
- **L607 EN**: Declares TableGen def record `M2_mpyd_acc_hh_s1`.
  **L607 CN**: 声明 TableGen def 记录 `M2_mpyd_acc_hh_s1`。
- **L608 EN**: Declares TableGen def record `M2_mpyd_acc_hl_s0`.
  **L608 CN**: 声明 TableGen def 记录 `M2_mpyd_acc_hl_s0`。
- **L609 EN**: Declares TableGen def record `M2_mpyd_acc_hl_s1`.
  **L609 CN**: 声明 TableGen def 记录 `M2_mpyd_acc_hl_s1`。
- **L610 EN**: Declares TableGen def record `M2_mpyd_acc_lh_s0`.
  **L610 CN**: 声明 TableGen def 记录 `M2_mpyd_acc_lh_s0`。
- **L611 EN**: Declares TableGen def record `M2_mpyd_acc_lh_s1`.
  **L611 CN**: 声明 TableGen def 记录 `M2_mpyd_acc_lh_s1`。
- **L612 EN**: Declares TableGen def record `M2_mpyd_acc_ll_s0`.
  **L612 CN**: 声明 TableGen def 记录 `M2_mpyd_acc_ll_s0`。
- **L613 EN**: Declares TableGen def record `M2_mpyd_acc_ll_s1`.
  **L613 CN**: 声明 TableGen def 记录 `M2_mpyd_acc_ll_s1`。
- **L614 EN**: Declares TableGen def record `M2_mpyd_hh_s0`.
  **L614 CN**: 声明 TableGen def 记录 `M2_mpyd_hh_s0`。
- **L615 EN**: Declares TableGen def record `M2_mpyd_hh_s1`.
  **L615 CN**: 声明 TableGen def 记录 `M2_mpyd_hh_s1`。
- **L616 EN**: Declares TableGen def record `M2_mpyd_hl_s0`.
  **L616 CN**: 声明 TableGen def 记录 `M2_mpyd_hl_s0`。

### Lines 617-644

````tablegen
def M2_mpyd_hl_s1 : HexagonBuiltin<"long long int(int, int)">;
def M2_mpyd_lh_s0 : HexagonBuiltin<"long long int(int, int)">;
def M2_mpyd_lh_s1 : HexagonBuiltin<"long long int(int, int)">;
def M2_mpyd_ll_s0 : HexagonBuiltin<"long long int(int, int)">;
def M2_mpyd_ll_s1 : HexagonBuiltin<"long long int(int, int)">;
def M2_mpyd_nac_hh_s0 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_mpyd_nac_hh_s1 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_mpyd_nac_hl_s0 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_mpyd_nac_hl_s1 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_mpyd_nac_lh_s0 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_mpyd_nac_lh_s1 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_mpyd_nac_ll_s0 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_mpyd_nac_ll_s1 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_mpyd_rnd_hh_s0 : HexagonBuiltin<"long long int(int, int)">;
def M2_mpyd_rnd_hh_s1 : HexagonBuiltin<"long long int(int, int)">;
def M2_mpyd_rnd_hl_s0 : HexagonBuiltin<"long long int(int, int)">;
def M2_mpyd_rnd_hl_s1 : HexagonBuiltin<"long long int(int, int)">;
def M2_mpyd_rnd_lh_s0 : HexagonBuiltin<"long long int(int, int)">;
def M2_mpyd_rnd_lh_s1 : HexagonBuiltin<"long long int(int, int)">;
def M2_mpyd_rnd_ll_s0 : HexagonBuiltin<"long long int(int, int)">;
def M2_mpyd_rnd_ll_s1 : HexagonBuiltin<"long long int(int, int)">;
def M2_mpyi : HexagonBuiltin<"int(int, int)">;
def M2_mpysmi : HexagonBuiltin<"int(int, _Constant int)">;
def M2_mpysu_up : HexagonBuiltin<"int(int, int)">;
def M2_mpyu_acc_hh_s0 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpyu_acc_hh_s1 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpyu_acc_hl_s0 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpyu_acc_hl_s1 : HexagonBuiltin<"int(int, int, int)">;
````
- **L617 EN**: Declares TableGen def record `M2_mpyd_hl_s1`.
  **L617 CN**: 声明 TableGen def 记录 `M2_mpyd_hl_s1`。
- **L618 EN**: Declares TableGen def record `M2_mpyd_lh_s0`.
  **L618 CN**: 声明 TableGen def 记录 `M2_mpyd_lh_s0`。
- **L619 EN**: Declares TableGen def record `M2_mpyd_lh_s1`.
  **L619 CN**: 声明 TableGen def 记录 `M2_mpyd_lh_s1`。
- **L620 EN**: Declares TableGen def record `M2_mpyd_ll_s0`.
  **L620 CN**: 声明 TableGen def 记录 `M2_mpyd_ll_s0`。
- **L621 EN**: Declares TableGen def record `M2_mpyd_ll_s1`.
  **L621 CN**: 声明 TableGen def 记录 `M2_mpyd_ll_s1`。
- **L622 EN**: Declares TableGen def record `M2_mpyd_nac_hh_s0`.
  **L622 CN**: 声明 TableGen def 记录 `M2_mpyd_nac_hh_s0`。
- **L623 EN**: Declares TableGen def record `M2_mpyd_nac_hh_s1`.
  **L623 CN**: 声明 TableGen def 记录 `M2_mpyd_nac_hh_s1`。
- **L624 EN**: Declares TableGen def record `M2_mpyd_nac_hl_s0`.
  **L624 CN**: 声明 TableGen def 记录 `M2_mpyd_nac_hl_s0`。
- **L625 EN**: Declares TableGen def record `M2_mpyd_nac_hl_s1`.
  **L625 CN**: 声明 TableGen def 记录 `M2_mpyd_nac_hl_s1`。
- **L626 EN**: Declares TableGen def record `M2_mpyd_nac_lh_s0`.
  **L626 CN**: 声明 TableGen def 记录 `M2_mpyd_nac_lh_s0`。
- **L627 EN**: Declares TableGen def record `M2_mpyd_nac_lh_s1`.
  **L627 CN**: 声明 TableGen def 记录 `M2_mpyd_nac_lh_s1`。
- **L628 EN**: Declares TableGen def record `M2_mpyd_nac_ll_s0`.
  **L628 CN**: 声明 TableGen def 记录 `M2_mpyd_nac_ll_s0`。
- **L629 EN**: Declares TableGen def record `M2_mpyd_nac_ll_s1`.
  **L629 CN**: 声明 TableGen def 记录 `M2_mpyd_nac_ll_s1`。
- **L630 EN**: Declares TableGen def record `M2_mpyd_rnd_hh_s0`.
  **L630 CN**: 声明 TableGen def 记录 `M2_mpyd_rnd_hh_s0`。
- **L631 EN**: Declares TableGen def record `M2_mpyd_rnd_hh_s1`.
  **L631 CN**: 声明 TableGen def 记录 `M2_mpyd_rnd_hh_s1`。
- **L632 EN**: Declares TableGen def record `M2_mpyd_rnd_hl_s0`.
  **L632 CN**: 声明 TableGen def 记录 `M2_mpyd_rnd_hl_s0`。
- **L633 EN**: Declares TableGen def record `M2_mpyd_rnd_hl_s1`.
  **L633 CN**: 声明 TableGen def 记录 `M2_mpyd_rnd_hl_s1`。
- **L634 EN**: Declares TableGen def record `M2_mpyd_rnd_lh_s0`.
  **L634 CN**: 声明 TableGen def 记录 `M2_mpyd_rnd_lh_s0`。
- **L635 EN**: Declares TableGen def record `M2_mpyd_rnd_lh_s1`.
  **L635 CN**: 声明 TableGen def 记录 `M2_mpyd_rnd_lh_s1`。
- **L636 EN**: Declares TableGen def record `M2_mpyd_rnd_ll_s0`.
  **L636 CN**: 声明 TableGen def 记录 `M2_mpyd_rnd_ll_s0`。
- **L637 EN**: Declares TableGen def record `M2_mpyd_rnd_ll_s1`.
  **L637 CN**: 声明 TableGen def 记录 `M2_mpyd_rnd_ll_s1`。
- **L638 EN**: Declares TableGen def record `M2_mpyi`.
  **L638 CN**: 声明 TableGen def 记录 `M2_mpyi`。
- **L639 EN**: Declares TableGen def record `M2_mpysmi`.
  **L639 CN**: 声明 TableGen def 记录 `M2_mpysmi`。
- **L640 EN**: Declares TableGen def record `M2_mpysu_up`.
  **L640 CN**: 声明 TableGen def 记录 `M2_mpysu_up`。
- **L641 EN**: Declares TableGen def record `M2_mpyu_acc_hh_s0`.
  **L641 CN**: 声明 TableGen def 记录 `M2_mpyu_acc_hh_s0`。
- **L642 EN**: Declares TableGen def record `M2_mpyu_acc_hh_s1`.
  **L642 CN**: 声明 TableGen def 记录 `M2_mpyu_acc_hh_s1`。
- **L643 EN**: Declares TableGen def record `M2_mpyu_acc_hl_s0`.
  **L643 CN**: 声明 TableGen def 记录 `M2_mpyu_acc_hl_s0`。
- **L644 EN**: Declares TableGen def record `M2_mpyu_acc_hl_s1`.
  **L644 CN**: 声明 TableGen def 记录 `M2_mpyu_acc_hl_s1`。

### Lines 645-672

````tablegen
def M2_mpyu_acc_lh_s0 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpyu_acc_lh_s1 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpyu_acc_ll_s0 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpyu_acc_ll_s1 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpyu_hh_s0 : HexagonBuiltin<"unsigned int(int, int)">;
def M2_mpyu_hh_s1 : HexagonBuiltin<"unsigned int(int, int)">;
def M2_mpyu_hl_s0 : HexagonBuiltin<"unsigned int(int, int)">;
def M2_mpyu_hl_s1 : HexagonBuiltin<"unsigned int(int, int)">;
def M2_mpyu_lh_s0 : HexagonBuiltin<"unsigned int(int, int)">;
def M2_mpyu_lh_s1 : HexagonBuiltin<"unsigned int(int, int)">;
def M2_mpyu_ll_s0 : HexagonBuiltin<"unsigned int(int, int)">;
def M2_mpyu_ll_s1 : HexagonBuiltin<"unsigned int(int, int)">;
def M2_mpyu_nac_hh_s0 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpyu_nac_hh_s1 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpyu_nac_hl_s0 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpyu_nac_hl_s1 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpyu_nac_lh_s0 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpyu_nac_lh_s1 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpyu_nac_ll_s0 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpyu_nac_ll_s1 : HexagonBuiltin<"int(int, int, int)">;
def M2_mpyu_up : HexagonBuiltin<"unsigned int(int, int)">;
def M2_mpyud_acc_hh_s0 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_mpyud_acc_hh_s1 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_mpyud_acc_hl_s0 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_mpyud_acc_hl_s1 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_mpyud_acc_lh_s0 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_mpyud_acc_lh_s1 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_mpyud_acc_ll_s0 : HexagonBuiltin<"long long int(long long int, int, int)">;
````
- **L645 EN**: Declares TableGen def record `M2_mpyu_acc_lh_s0`.
  **L645 CN**: 声明 TableGen def 记录 `M2_mpyu_acc_lh_s0`。
- **L646 EN**: Declares TableGen def record `M2_mpyu_acc_lh_s1`.
  **L646 CN**: 声明 TableGen def 记录 `M2_mpyu_acc_lh_s1`。
- **L647 EN**: Declares TableGen def record `M2_mpyu_acc_ll_s0`.
  **L647 CN**: 声明 TableGen def 记录 `M2_mpyu_acc_ll_s0`。
- **L648 EN**: Declares TableGen def record `M2_mpyu_acc_ll_s1`.
  **L648 CN**: 声明 TableGen def 记录 `M2_mpyu_acc_ll_s1`。
- **L649 EN**: Declares TableGen def record `M2_mpyu_hh_s0`.
  **L649 CN**: 声明 TableGen def 记录 `M2_mpyu_hh_s0`。
- **L650 EN**: Declares TableGen def record `M2_mpyu_hh_s1`.
  **L650 CN**: 声明 TableGen def 记录 `M2_mpyu_hh_s1`。
- **L651 EN**: Declares TableGen def record `M2_mpyu_hl_s0`.
  **L651 CN**: 声明 TableGen def 记录 `M2_mpyu_hl_s0`。
- **L652 EN**: Declares TableGen def record `M2_mpyu_hl_s1`.
  **L652 CN**: 声明 TableGen def 记录 `M2_mpyu_hl_s1`。
- **L653 EN**: Declares TableGen def record `M2_mpyu_lh_s0`.
  **L653 CN**: 声明 TableGen def 记录 `M2_mpyu_lh_s0`。
- **L654 EN**: Declares TableGen def record `M2_mpyu_lh_s1`.
  **L654 CN**: 声明 TableGen def 记录 `M2_mpyu_lh_s1`。
- **L655 EN**: Declares TableGen def record `M2_mpyu_ll_s0`.
  **L655 CN**: 声明 TableGen def 记录 `M2_mpyu_ll_s0`。
- **L656 EN**: Declares TableGen def record `M2_mpyu_ll_s1`.
  **L656 CN**: 声明 TableGen def 记录 `M2_mpyu_ll_s1`。
- **L657 EN**: Declares TableGen def record `M2_mpyu_nac_hh_s0`.
  **L657 CN**: 声明 TableGen def 记录 `M2_mpyu_nac_hh_s0`。
- **L658 EN**: Declares TableGen def record `M2_mpyu_nac_hh_s1`.
  **L658 CN**: 声明 TableGen def 记录 `M2_mpyu_nac_hh_s1`。
- **L659 EN**: Declares TableGen def record `M2_mpyu_nac_hl_s0`.
  **L659 CN**: 声明 TableGen def 记录 `M2_mpyu_nac_hl_s0`。
- **L660 EN**: Declares TableGen def record `M2_mpyu_nac_hl_s1`.
  **L660 CN**: 声明 TableGen def 记录 `M2_mpyu_nac_hl_s1`。
- **L661 EN**: Declares TableGen def record `M2_mpyu_nac_lh_s0`.
  **L661 CN**: 声明 TableGen def 记录 `M2_mpyu_nac_lh_s0`。
- **L662 EN**: Declares TableGen def record `M2_mpyu_nac_lh_s1`.
  **L662 CN**: 声明 TableGen def 记录 `M2_mpyu_nac_lh_s1`。
- **L663 EN**: Declares TableGen def record `M2_mpyu_nac_ll_s0`.
  **L663 CN**: 声明 TableGen def 记录 `M2_mpyu_nac_ll_s0`。
- **L664 EN**: Declares TableGen def record `M2_mpyu_nac_ll_s1`.
  **L664 CN**: 声明 TableGen def 记录 `M2_mpyu_nac_ll_s1`。
- **L665 EN**: Declares TableGen def record `M2_mpyu_up`.
  **L665 CN**: 声明 TableGen def 记录 `M2_mpyu_up`。
- **L666 EN**: Declares TableGen def record `M2_mpyud_acc_hh_s0`.
  **L666 CN**: 声明 TableGen def 记录 `M2_mpyud_acc_hh_s0`。
- **L667 EN**: Declares TableGen def record `M2_mpyud_acc_hh_s1`.
  **L667 CN**: 声明 TableGen def 记录 `M2_mpyud_acc_hh_s1`。
- **L668 EN**: Declares TableGen def record `M2_mpyud_acc_hl_s0`.
  **L668 CN**: 声明 TableGen def 记录 `M2_mpyud_acc_hl_s0`。
- **L669 EN**: Declares TableGen def record `M2_mpyud_acc_hl_s1`.
  **L669 CN**: 声明 TableGen def 记录 `M2_mpyud_acc_hl_s1`。
- **L670 EN**: Declares TableGen def record `M2_mpyud_acc_lh_s0`.
  **L670 CN**: 声明 TableGen def 记录 `M2_mpyud_acc_lh_s0`。
- **L671 EN**: Declares TableGen def record `M2_mpyud_acc_lh_s1`.
  **L671 CN**: 声明 TableGen def 记录 `M2_mpyud_acc_lh_s1`。
- **L672 EN**: Declares TableGen def record `M2_mpyud_acc_ll_s0`.
  **L672 CN**: 声明 TableGen def 记录 `M2_mpyud_acc_ll_s0`。

### Lines 673-700

````tablegen
def M2_mpyud_acc_ll_s1 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_mpyud_hh_s0 : HexagonBuiltin<"unsigned long long int(int, int)">;
def M2_mpyud_hh_s1 : HexagonBuiltin<"unsigned long long int(int, int)">;
def M2_mpyud_hl_s0 : HexagonBuiltin<"unsigned long long int(int, int)">;
def M2_mpyud_hl_s1 : HexagonBuiltin<"unsigned long long int(int, int)">;
def M2_mpyud_lh_s0 : HexagonBuiltin<"unsigned long long int(int, int)">;
def M2_mpyud_lh_s1 : HexagonBuiltin<"unsigned long long int(int, int)">;
def M2_mpyud_ll_s0 : HexagonBuiltin<"unsigned long long int(int, int)">;
def M2_mpyud_ll_s1 : HexagonBuiltin<"unsigned long long int(int, int)">;
def M2_mpyud_nac_hh_s0 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_mpyud_nac_hh_s1 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_mpyud_nac_hl_s0 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_mpyud_nac_hl_s1 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_mpyud_nac_lh_s0 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_mpyud_nac_lh_s1 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_mpyud_nac_ll_s0 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_mpyud_nac_ll_s1 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_mpyui : HexagonBuiltin<"int(int, int)">;
def M2_nacci : HexagonBuiltin<"int(int, int, int)">;
def M2_naccii : HexagonBuiltin<"int(int, int, _Constant int)">;
def M2_subacc : HexagonBuiltin<"int(int, int, int)">;
def M2_vabsdiffh : HexagonBuiltin<"long long int(long long int, long long int)">;
def M2_vabsdiffw : HexagonBuiltin<"long long int(long long int, long long int)">;
def M2_vcmac_s0_sat_i : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
def M2_vcmac_s0_sat_r : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
def M2_vcmpy_s0_sat_i : HexagonBuiltin<"long long int(long long int, long long int)">;
def M2_vcmpy_s0_sat_r : HexagonBuiltin<"long long int(long long int, long long int)">;
def M2_vcmpy_s1_sat_i : HexagonBuiltin<"long long int(long long int, long long int)">;
````
- **L673 EN**: Declares TableGen def record `M2_mpyud_acc_ll_s1`.
  **L673 CN**: 声明 TableGen def 记录 `M2_mpyud_acc_ll_s1`。
- **L674 EN**: Declares TableGen def record `M2_mpyud_hh_s0`.
  **L674 CN**: 声明 TableGen def 记录 `M2_mpyud_hh_s0`。
- **L675 EN**: Declares TableGen def record `M2_mpyud_hh_s1`.
  **L675 CN**: 声明 TableGen def 记录 `M2_mpyud_hh_s1`。
- **L676 EN**: Declares TableGen def record `M2_mpyud_hl_s0`.
  **L676 CN**: 声明 TableGen def 记录 `M2_mpyud_hl_s0`。
- **L677 EN**: Declares TableGen def record `M2_mpyud_hl_s1`.
  **L677 CN**: 声明 TableGen def 记录 `M2_mpyud_hl_s1`。
- **L678 EN**: Declares TableGen def record `M2_mpyud_lh_s0`.
  **L678 CN**: 声明 TableGen def 记录 `M2_mpyud_lh_s0`。
- **L679 EN**: Declares TableGen def record `M2_mpyud_lh_s1`.
  **L679 CN**: 声明 TableGen def 记录 `M2_mpyud_lh_s1`。
- **L680 EN**: Declares TableGen def record `M2_mpyud_ll_s0`.
  **L680 CN**: 声明 TableGen def 记录 `M2_mpyud_ll_s0`。
- **L681 EN**: Declares TableGen def record `M2_mpyud_ll_s1`.
  **L681 CN**: 声明 TableGen def 记录 `M2_mpyud_ll_s1`。
- **L682 EN**: Declares TableGen def record `M2_mpyud_nac_hh_s0`.
  **L682 CN**: 声明 TableGen def 记录 `M2_mpyud_nac_hh_s0`。
- **L683 EN**: Declares TableGen def record `M2_mpyud_nac_hh_s1`.
  **L683 CN**: 声明 TableGen def 记录 `M2_mpyud_nac_hh_s1`。
- **L684 EN**: Declares TableGen def record `M2_mpyud_nac_hl_s0`.
  **L684 CN**: 声明 TableGen def 记录 `M2_mpyud_nac_hl_s0`。
- **L685 EN**: Declares TableGen def record `M2_mpyud_nac_hl_s1`.
  **L685 CN**: 声明 TableGen def 记录 `M2_mpyud_nac_hl_s1`。
- **L686 EN**: Declares TableGen def record `M2_mpyud_nac_lh_s0`.
  **L686 CN**: 声明 TableGen def 记录 `M2_mpyud_nac_lh_s0`。
- **L687 EN**: Declares TableGen def record `M2_mpyud_nac_lh_s1`.
  **L687 CN**: 声明 TableGen def 记录 `M2_mpyud_nac_lh_s1`。
- **L688 EN**: Declares TableGen def record `M2_mpyud_nac_ll_s0`.
  **L688 CN**: 声明 TableGen def 记录 `M2_mpyud_nac_ll_s0`。
- **L689 EN**: Declares TableGen def record `M2_mpyud_nac_ll_s1`.
  **L689 CN**: 声明 TableGen def 记录 `M2_mpyud_nac_ll_s1`。
- **L690 EN**: Declares TableGen def record `M2_mpyui`.
  **L690 CN**: 声明 TableGen def 记录 `M2_mpyui`。
- **L691 EN**: Declares TableGen def record `M2_nacci`.
  **L691 CN**: 声明 TableGen def 记录 `M2_nacci`。
- **L692 EN**: Declares TableGen def record `M2_naccii`.
  **L692 CN**: 声明 TableGen def 记录 `M2_naccii`。
- **L693 EN**: Declares TableGen def record `M2_subacc`.
  **L693 CN**: 声明 TableGen def 记录 `M2_subacc`。
- **L694 EN**: Declares TableGen def record `M2_vabsdiffh`.
  **L694 CN**: 声明 TableGen def 记录 `M2_vabsdiffh`。
- **L695 EN**: Declares TableGen def record `M2_vabsdiffw`.
  **L695 CN**: 声明 TableGen def 记录 `M2_vabsdiffw`。
- **L696 EN**: Declares TableGen def record `M2_vcmac_s0_sat_i`.
  **L696 CN**: 声明 TableGen def 记录 `M2_vcmac_s0_sat_i`。
- **L697 EN**: Declares TableGen def record `M2_vcmac_s0_sat_r`.
  **L697 CN**: 声明 TableGen def 记录 `M2_vcmac_s0_sat_r`。
- **L698 EN**: Declares TableGen def record `M2_vcmpy_s0_sat_i`.
  **L698 CN**: 声明 TableGen def 记录 `M2_vcmpy_s0_sat_i`。
- **L699 EN**: Declares TableGen def record `M2_vcmpy_s0_sat_r`.
  **L699 CN**: 声明 TableGen def 记录 `M2_vcmpy_s0_sat_r`。
- **L700 EN**: Declares TableGen def record `M2_vcmpy_s1_sat_i`.
  **L700 CN**: 声明 TableGen def 记录 `M2_vcmpy_s1_sat_i`。

### Lines 701-728

````tablegen
def M2_vcmpy_s1_sat_r : HexagonBuiltin<"long long int(long long int, long long int)">;
def M2_vdmacs_s0 : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
def M2_vdmacs_s1 : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
def M2_vdmpyrs_s0 : HexagonBuiltin<"int(long long int, long long int)">;
def M2_vdmpyrs_s1 : HexagonBuiltin<"int(long long int, long long int)">;
def M2_vdmpys_s0 : HexagonBuiltin<"long long int(long long int, long long int)">;
def M2_vdmpys_s1 : HexagonBuiltin<"long long int(long long int, long long int)">;
def M2_vmac2 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_vmac2es : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
def M2_vmac2es_s0 : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
def M2_vmac2es_s1 : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
def M2_vmac2s_s0 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_vmac2s_s1 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_vmac2su_s0 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_vmac2su_s1 : HexagonBuiltin<"long long int(long long int, int, int)">;
def M2_vmpy2es_s0 : HexagonBuiltin<"long long int(long long int, long long int)">;
def M2_vmpy2es_s1 : HexagonBuiltin<"long long int(long long int, long long int)">;
def M2_vmpy2s_s0 : HexagonBuiltin<"long long int(int, int)">;
def M2_vmpy2s_s0pack : HexagonBuiltin<"int(int, int)">;
def M2_vmpy2s_s1 : HexagonBuiltin<"long long int(int, int)">;
def M2_vmpy2s_s1pack : HexagonBuiltin<"int(int, int)">;
def M2_vmpy2su_s0 : HexagonBuiltin<"long long int(int, int)">;
def M2_vmpy2su_s1 : HexagonBuiltin<"long long int(int, int)">;
def M2_vraddh : HexagonBuiltin<"int(long long int, long long int)">;
def M2_vradduh : HexagonBuiltin<"int(long long int, long long int)">;
def M2_vrcmaci_s0 : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
def M2_vrcmaci_s0c : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
def M2_vrcmacr_s0 : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
````
- **L701 EN**: Declares TableGen def record `M2_vcmpy_s1_sat_r`.
  **L701 CN**: 声明 TableGen def 记录 `M2_vcmpy_s1_sat_r`。
- **L702 EN**: Declares TableGen def record `M2_vdmacs_s0`.
  **L702 CN**: 声明 TableGen def 记录 `M2_vdmacs_s0`。
- **L703 EN**: Declares TableGen def record `M2_vdmacs_s1`.
  **L703 CN**: 声明 TableGen def 记录 `M2_vdmacs_s1`。
- **L704 EN**: Declares TableGen def record `M2_vdmpyrs_s0`.
  **L704 CN**: 声明 TableGen def 记录 `M2_vdmpyrs_s0`。
- **L705 EN**: Declares TableGen def record `M2_vdmpyrs_s1`.
  **L705 CN**: 声明 TableGen def 记录 `M2_vdmpyrs_s1`。
- **L706 EN**: Declares TableGen def record `M2_vdmpys_s0`.
  **L706 CN**: 声明 TableGen def 记录 `M2_vdmpys_s0`。
- **L707 EN**: Declares TableGen def record `M2_vdmpys_s1`.
  **L707 CN**: 声明 TableGen def 记录 `M2_vdmpys_s1`。
- **L708 EN**: Declares TableGen def record `M2_vmac2`.
  **L708 CN**: 声明 TableGen def 记录 `M2_vmac2`。
- **L709 EN**: Declares TableGen def record `M2_vmac2es`.
  **L709 CN**: 声明 TableGen def 记录 `M2_vmac2es`。
- **L710 EN**: Declares TableGen def record `M2_vmac2es_s0`.
  **L710 CN**: 声明 TableGen def 记录 `M2_vmac2es_s0`。
- **L711 EN**: Declares TableGen def record `M2_vmac2es_s1`.
  **L711 CN**: 声明 TableGen def 记录 `M2_vmac2es_s1`。
- **L712 EN**: Declares TableGen def record `M2_vmac2s_s0`.
  **L712 CN**: 声明 TableGen def 记录 `M2_vmac2s_s0`。
- **L713 EN**: Declares TableGen def record `M2_vmac2s_s1`.
  **L713 CN**: 声明 TableGen def 记录 `M2_vmac2s_s1`。
- **L714 EN**: Declares TableGen def record `M2_vmac2su_s0`.
  **L714 CN**: 声明 TableGen def 记录 `M2_vmac2su_s0`。
- **L715 EN**: Declares TableGen def record `M2_vmac2su_s1`.
  **L715 CN**: 声明 TableGen def 记录 `M2_vmac2su_s1`。
- **L716 EN**: Declares TableGen def record `M2_vmpy2es_s0`.
  **L716 CN**: 声明 TableGen def 记录 `M2_vmpy2es_s0`。
- **L717 EN**: Declares TableGen def record `M2_vmpy2es_s1`.
  **L717 CN**: 声明 TableGen def 记录 `M2_vmpy2es_s1`。
- **L718 EN**: Declares TableGen def record `M2_vmpy2s_s0`.
  **L718 CN**: 声明 TableGen def 记录 `M2_vmpy2s_s0`。
- **L719 EN**: Declares TableGen def record `M2_vmpy2s_s0pack`.
  **L719 CN**: 声明 TableGen def 记录 `M2_vmpy2s_s0pack`。
- **L720 EN**: Declares TableGen def record `M2_vmpy2s_s1`.
  **L720 CN**: 声明 TableGen def 记录 `M2_vmpy2s_s1`。
- **L721 EN**: Declares TableGen def record `M2_vmpy2s_s1pack`.
  **L721 CN**: 声明 TableGen def 记录 `M2_vmpy2s_s1pack`。
- **L722 EN**: Declares TableGen def record `M2_vmpy2su_s0`.
  **L722 CN**: 声明 TableGen def 记录 `M2_vmpy2su_s0`。
- **L723 EN**: Declares TableGen def record `M2_vmpy2su_s1`.
  **L723 CN**: 声明 TableGen def 记录 `M2_vmpy2su_s1`。
- **L724 EN**: Declares TableGen def record `M2_vraddh`.
  **L724 CN**: 声明 TableGen def 记录 `M2_vraddh`。
- **L725 EN**: Declares TableGen def record `M2_vradduh`.
  **L725 CN**: 声明 TableGen def 记录 `M2_vradduh`。
- **L726 EN**: Declares TableGen def record `M2_vrcmaci_s0`.
  **L726 CN**: 声明 TableGen def 记录 `M2_vrcmaci_s0`。
- **L727 EN**: Declares TableGen def record `M2_vrcmaci_s0c`.
  **L727 CN**: 声明 TableGen def 记录 `M2_vrcmaci_s0c`。
- **L728 EN**: Declares TableGen def record `M2_vrcmacr_s0`.
  **L728 CN**: 声明 TableGen def 记录 `M2_vrcmacr_s0`。

### Lines 729-756

````tablegen
def M2_vrcmacr_s0c : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
def M2_vrcmpyi_s0 : HexagonBuiltin<"long long int(long long int, long long int)">;
def M2_vrcmpyi_s0c : HexagonBuiltin<"long long int(long long int, long long int)">;
def M2_vrcmpyr_s0 : HexagonBuiltin<"long long int(long long int, long long int)">;
def M2_vrcmpyr_s0c : HexagonBuiltin<"long long int(long long int, long long int)">;
def M2_vrcmpys_acc_s1 : HexagonBuiltin<"long long int(long long int, long long int, int)">;
def M2_vrcmpys_s1 : HexagonBuiltin<"long long int(long long int, int)">;
def M2_vrcmpys_s1rp : HexagonBuiltin<"int(long long int, int)">;
def M2_vrmac_s0 : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
def M2_vrmpy_s0 : HexagonBuiltin<"long long int(long long int, long long int)">;
def M2_xor_xacc : HexagonBuiltin<"int(int, int, int)">;
def M4_and_and : HexagonBuiltin<"int(int, int, int)">;
def M4_and_andn : HexagonBuiltin<"int(int, int, int)">;
def M4_and_or : HexagonBuiltin<"int(int, int, int)">;
def M4_and_xor : HexagonBuiltin<"int(int, int, int)">;
def M4_cmpyi_wh : HexagonBuiltin<"int(long long int, int)">;
def M4_cmpyi_whc : HexagonBuiltin<"int(long long int, int)">;
def M4_cmpyr_wh : HexagonBuiltin<"int(long long int, int)">;
def M4_cmpyr_whc : HexagonBuiltin<"int(long long int, int)">;
def M4_mac_up_s1_sat : HexagonBuiltin<"int(int, int, int)">;
def M4_mpyri_addi : HexagonBuiltin<"int(unsigned _Constant int, int, unsigned _Constant int)">;
def M4_mpyri_addr : HexagonBuiltin<"int(int, int, unsigned _Constant int)">;
def M4_mpyri_addr_u2 : HexagonBuiltin<"int(int, unsigned _Constant int, int)">;
def M4_mpyrr_addi : HexagonBuiltin<"int(unsigned _Constant int, int, int)">;
def M4_mpyrr_addr : HexagonBuiltin<"int(int, int, int)">;
def M4_nac_up_s1_sat : HexagonBuiltin<"int(int, int, int)">;
def M4_or_and : HexagonBuiltin<"int(int, int, int)">;
def M4_or_andn : HexagonBuiltin<"int(int, int, int)">;
````
- **L729 EN**: Declares TableGen def record `M2_vrcmacr_s0c`.
  **L729 CN**: 声明 TableGen def 记录 `M2_vrcmacr_s0c`。
- **L730 EN**: Declares TableGen def record `M2_vrcmpyi_s0`.
  **L730 CN**: 声明 TableGen def 记录 `M2_vrcmpyi_s0`。
- **L731 EN**: Declares TableGen def record `M2_vrcmpyi_s0c`.
  **L731 CN**: 声明 TableGen def 记录 `M2_vrcmpyi_s0c`。
- **L732 EN**: Declares TableGen def record `M2_vrcmpyr_s0`.
  **L732 CN**: 声明 TableGen def 记录 `M2_vrcmpyr_s0`。
- **L733 EN**: Declares TableGen def record `M2_vrcmpyr_s0c`.
  **L733 CN**: 声明 TableGen def 记录 `M2_vrcmpyr_s0c`。
- **L734 EN**: Declares TableGen def record `M2_vrcmpys_acc_s1`.
  **L734 CN**: 声明 TableGen def 记录 `M2_vrcmpys_acc_s1`。
- **L735 EN**: Declares TableGen def record `M2_vrcmpys_s1`.
  **L735 CN**: 声明 TableGen def 记录 `M2_vrcmpys_s1`。
- **L736 EN**: Declares TableGen def record `M2_vrcmpys_s1rp`.
  **L736 CN**: 声明 TableGen def 记录 `M2_vrcmpys_s1rp`。
- **L737 EN**: Declares TableGen def record `M2_vrmac_s0`.
  **L737 CN**: 声明 TableGen def 记录 `M2_vrmac_s0`。
- **L738 EN**: Declares TableGen def record `M2_vrmpy_s0`.
  **L738 CN**: 声明 TableGen def 记录 `M2_vrmpy_s0`。
- **L739 EN**: Declares TableGen def record `M2_xor_xacc`.
  **L739 CN**: 声明 TableGen def 记录 `M2_xor_xacc`。
- **L740 EN**: Declares TableGen def record `M4_and_and`.
  **L740 CN**: 声明 TableGen def 记录 `M4_and_and`。
- **L741 EN**: Declares TableGen def record `M4_and_andn`.
  **L741 CN**: 声明 TableGen def 记录 `M4_and_andn`。
- **L742 EN**: Declares TableGen def record `M4_and_or`.
  **L742 CN**: 声明 TableGen def 记录 `M4_and_or`。
- **L743 EN**: Declares TableGen def record `M4_and_xor`.
  **L743 CN**: 声明 TableGen def 记录 `M4_and_xor`。
- **L744 EN**: Declares TableGen def record `M4_cmpyi_wh`.
  **L744 CN**: 声明 TableGen def 记录 `M4_cmpyi_wh`。
- **L745 EN**: Declares TableGen def record `M4_cmpyi_whc`.
  **L745 CN**: 声明 TableGen def 记录 `M4_cmpyi_whc`。
- **L746 EN**: Declares TableGen def record `M4_cmpyr_wh`.
  **L746 CN**: 声明 TableGen def 记录 `M4_cmpyr_wh`。
- **L747 EN**: Declares TableGen def record `M4_cmpyr_whc`.
  **L747 CN**: 声明 TableGen def 记录 `M4_cmpyr_whc`。
- **L748 EN**: Declares TableGen def record `M4_mac_up_s1_sat`.
  **L748 CN**: 声明 TableGen def 记录 `M4_mac_up_s1_sat`。
- **L749 EN**: Declares TableGen def record `M4_mpyri_addi`.
  **L749 CN**: 声明 TableGen def 记录 `M4_mpyri_addi`。
- **L750 EN**: Declares TableGen def record `M4_mpyri_addr`.
  **L750 CN**: 声明 TableGen def 记录 `M4_mpyri_addr`。
- **L751 EN**: Declares TableGen def record `M4_mpyri_addr_u2`.
  **L751 CN**: 声明 TableGen def 记录 `M4_mpyri_addr_u2`。
- **L752 EN**: Declares TableGen def record `M4_mpyrr_addi`.
  **L752 CN**: 声明 TableGen def 记录 `M4_mpyrr_addi`。
- **L753 EN**: Declares TableGen def record `M4_mpyrr_addr`.
  **L753 CN**: 声明 TableGen def 记录 `M4_mpyrr_addr`。
- **L754 EN**: Declares TableGen def record `M4_nac_up_s1_sat`.
  **L754 CN**: 声明 TableGen def 记录 `M4_nac_up_s1_sat`。
- **L755 EN**: Declares TableGen def record `M4_or_and`.
  **L755 CN**: 声明 TableGen def 记录 `M4_or_and`。
- **L756 EN**: Declares TableGen def record `M4_or_andn`.
  **L756 CN**: 声明 TableGen def 记录 `M4_or_andn`。

### Lines 757-784

````tablegen
def M4_or_or : HexagonBuiltin<"int(int, int, int)">;
def M4_or_xor : HexagonBuiltin<"int(int, int, int)">;
def M4_pmpyw : HexagonBuiltin<"long long int(int, int)">;
def M4_pmpyw_acc : HexagonBuiltin<"long long int(long long int, int, int)">;
def M4_vpmpyh : HexagonBuiltin<"long long int(int, int)">;
def M4_vpmpyh_acc : HexagonBuiltin<"long long int(long long int, int, int)">;
def M4_vrmpyeh_acc_s0 : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
def M4_vrmpyeh_acc_s1 : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
def M4_vrmpyeh_s0 : HexagonBuiltin<"long long int(long long int, long long int)">;
def M4_vrmpyeh_s1 : HexagonBuiltin<"long long int(long long int, long long int)">;
def M4_vrmpyoh_acc_s0 : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
def M4_vrmpyoh_acc_s1 : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
def M4_vrmpyoh_s0 : HexagonBuiltin<"long long int(long long int, long long int)">;
def M4_vrmpyoh_s1 : HexagonBuiltin<"long long int(long long int, long long int)">;
def M4_xor_and : HexagonBuiltin<"int(int, int, int)">;
def M4_xor_andn : HexagonBuiltin<"int(int, int, int)">;
def M4_xor_or : HexagonBuiltin<"int(int, int, int)">;
def M4_xor_xacc : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
def M5_vdmacbsu : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
def M5_vdmpybsu : HexagonBuiltin<"long long int(long long int, long long int)">;
def M5_vmacbsu : HexagonBuiltin<"long long int(long long int, int, int)">;
def M5_vmacbuu : HexagonBuiltin<"long long int(long long int, int, int)">;
def M5_vmpybsu : HexagonBuiltin<"long long int(int, int)">;
def M5_vmpybuu : HexagonBuiltin<"long long int(int, int)">;
def M5_vrmacbsu : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
def M5_vrmacbuu : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
def M5_vrmpybsu : HexagonBuiltin<"long long int(long long int, long long int)">;
def M5_vrmpybuu : HexagonBuiltin<"long long int(long long int, long long int)">;
````
- **L757 EN**: Declares TableGen def record `M4_or_or`.
  **L757 CN**: 声明 TableGen def 记录 `M4_or_or`。
- **L758 EN**: Declares TableGen def record `M4_or_xor`.
  **L758 CN**: 声明 TableGen def 记录 `M4_or_xor`。
- **L759 EN**: Declares TableGen def record `M4_pmpyw`.
  **L759 CN**: 声明 TableGen def 记录 `M4_pmpyw`。
- **L760 EN**: Declares TableGen def record `M4_pmpyw_acc`.
  **L760 CN**: 声明 TableGen def 记录 `M4_pmpyw_acc`。
- **L761 EN**: Declares TableGen def record `M4_vpmpyh`.
  **L761 CN**: 声明 TableGen def 记录 `M4_vpmpyh`。
- **L762 EN**: Declares TableGen def record `M4_vpmpyh_acc`.
  **L762 CN**: 声明 TableGen def 记录 `M4_vpmpyh_acc`。
- **L763 EN**: Declares TableGen def record `M4_vrmpyeh_acc_s0`.
  **L763 CN**: 声明 TableGen def 记录 `M4_vrmpyeh_acc_s0`。
- **L764 EN**: Declares TableGen def record `M4_vrmpyeh_acc_s1`.
  **L764 CN**: 声明 TableGen def 记录 `M4_vrmpyeh_acc_s1`。
- **L765 EN**: Declares TableGen def record `M4_vrmpyeh_s0`.
  **L765 CN**: 声明 TableGen def 记录 `M4_vrmpyeh_s0`。
- **L766 EN**: Declares TableGen def record `M4_vrmpyeh_s1`.
  **L766 CN**: 声明 TableGen def 记录 `M4_vrmpyeh_s1`。
- **L767 EN**: Declares TableGen def record `M4_vrmpyoh_acc_s0`.
  **L767 CN**: 声明 TableGen def 记录 `M4_vrmpyoh_acc_s0`。
- **L768 EN**: Declares TableGen def record `M4_vrmpyoh_acc_s1`.
  **L768 CN**: 声明 TableGen def 记录 `M4_vrmpyoh_acc_s1`。
- **L769 EN**: Declares TableGen def record `M4_vrmpyoh_s0`.
  **L769 CN**: 声明 TableGen def 记录 `M4_vrmpyoh_s0`。
- **L770 EN**: Declares TableGen def record `M4_vrmpyoh_s1`.
  **L770 CN**: 声明 TableGen def 记录 `M4_vrmpyoh_s1`。
- **L771 EN**: Declares TableGen def record `M4_xor_and`.
  **L771 CN**: 声明 TableGen def 记录 `M4_xor_and`。
- **L772 EN**: Declares TableGen def record `M4_xor_andn`.
  **L772 CN**: 声明 TableGen def 记录 `M4_xor_andn`。
- **L773 EN**: Declares TableGen def record `M4_xor_or`.
  **L773 CN**: 声明 TableGen def 记录 `M4_xor_or`。
- **L774 EN**: Declares TableGen def record `M4_xor_xacc`.
  **L774 CN**: 声明 TableGen def 记录 `M4_xor_xacc`。
- **L775 EN**: Declares TableGen def record `M5_vdmacbsu`.
  **L775 CN**: 声明 TableGen def 记录 `M5_vdmacbsu`。
- **L776 EN**: Declares TableGen def record `M5_vdmpybsu`.
  **L776 CN**: 声明 TableGen def 记录 `M5_vdmpybsu`。
- **L777 EN**: Declares TableGen def record `M5_vmacbsu`.
  **L777 CN**: 声明 TableGen def 记录 `M5_vmacbsu`。
- **L778 EN**: Declares TableGen def record `M5_vmacbuu`.
  **L778 CN**: 声明 TableGen def 记录 `M5_vmacbuu`。
- **L779 EN**: Declares TableGen def record `M5_vmpybsu`.
  **L779 CN**: 声明 TableGen def 记录 `M5_vmpybsu`。
- **L780 EN**: Declares TableGen def record `M5_vmpybuu`.
  **L780 CN**: 声明 TableGen def 记录 `M5_vmpybuu`。
- **L781 EN**: Declares TableGen def record `M5_vrmacbsu`.
  **L781 CN**: 声明 TableGen def 记录 `M5_vrmacbsu`。
- **L782 EN**: Declares TableGen def record `M5_vrmacbuu`.
  **L782 CN**: 声明 TableGen def 记录 `M5_vrmacbuu`。
- **L783 EN**: Declares TableGen def record `M5_vrmpybsu`.
  **L783 CN**: 声明 TableGen def 记录 `M5_vrmpybsu`。
- **L784 EN**: Declares TableGen def record `M5_vrmpybuu`.
  **L784 CN**: 声明 TableGen def 记录 `M5_vrmpybuu`。

### Lines 785-812

````tablegen
def S2_addasl_rrri : HexagonBuiltin<"int(int, int, unsigned _Constant int)">;
def S2_asl_i_p : HexagonBuiltin<"long long int(long long int, unsigned _Constant int)">;
def S2_asl_i_p_acc : HexagonBuiltin<"long long int(long long int, long long int, unsigned _Constant int)">;
def S2_asl_i_p_and : HexagonBuiltin<"long long int(long long int, long long int, unsigned _Constant int)">;
def S2_asl_i_p_nac : HexagonBuiltin<"long long int(long long int, long long int, unsigned _Constant int)">;
def S2_asl_i_p_or : HexagonBuiltin<"long long int(long long int, long long int, unsigned _Constant int)">;
def S2_asl_i_p_xacc : HexagonBuiltin<"long long int(long long int, long long int, unsigned _Constant int)">;
def S2_asl_i_r : HexagonBuiltin<"int(int, unsigned _Constant int)">;
def S2_asl_i_r_acc : HexagonBuiltin<"int(int, int, unsigned _Constant int)">;
def S2_asl_i_r_and : HexagonBuiltin<"int(int, int, unsigned _Constant int)">;
def S2_asl_i_r_nac : HexagonBuiltin<"int(int, int, unsigned _Constant int)">;
def S2_asl_i_r_or : HexagonBuiltin<"int(int, int, unsigned _Constant int)">;
def S2_asl_i_r_sat : HexagonBuiltin<"int(int, unsigned _Constant int)">;
def S2_asl_i_r_xacc : HexagonBuiltin<"int(int, int, unsigned _Constant int)">;
def S2_asl_i_vh : HexagonBuiltin<"long long int(long long int, unsigned _Constant int)">;
def S2_asl_i_vw : HexagonBuiltin<"long long int(long long int, unsigned _Constant int)">;
def S2_asl_r_p : HexagonBuiltin<"long long int(long long int, int)">;
def S2_asl_r_p_acc : HexagonBuiltin<"long long int(long long int, long long int, int)">;
def S2_asl_r_p_and : HexagonBuiltin<"long long int(long long int, long long int, int)">;
def S2_asl_r_p_nac : HexagonBuiltin<"long long int(long long int, long long int, int)">;
def S2_asl_r_p_or : HexagonBuiltin<"long long int(long long int, long long int, int)">;
def S2_asl_r_p_xor : HexagonBuiltin<"long long int(long long int, long long int, int)">;
def S2_asl_r_r : HexagonBuiltin<"int(int, int)">;
def S2_asl_r_r_acc : HexagonBuiltin<"int(int, int, int)">;
def S2_asl_r_r_and : HexagonBuiltin<"int(int, int, int)">;
def S2_asl_r_r_nac : HexagonBuiltin<"int(int, int, int)">;
def S2_asl_r_r_or : HexagonBuiltin<"int(int, int, int)">;
def S2_asl_r_r_sat : HexagonBuiltin<"int(int, int)">;
````
- **L785 EN**: Declares TableGen def record `S2_addasl_rrri`.
  **L785 CN**: 声明 TableGen def 记录 `S2_addasl_rrri`。
- **L786 EN**: Declares TableGen def record `S2_asl_i_p`.
  **L786 CN**: 声明 TableGen def 记录 `S2_asl_i_p`。
- **L787 EN**: Declares TableGen def record `S2_asl_i_p_acc`.
  **L787 CN**: 声明 TableGen def 记录 `S2_asl_i_p_acc`。
- **L788 EN**: Declares TableGen def record `S2_asl_i_p_and`.
  **L788 CN**: 声明 TableGen def 记录 `S2_asl_i_p_and`。
- **L789 EN**: Declares TableGen def record `S2_asl_i_p_nac`.
  **L789 CN**: 声明 TableGen def 记录 `S2_asl_i_p_nac`。
- **L790 EN**: Declares TableGen def record `S2_asl_i_p_or`.
  **L790 CN**: 声明 TableGen def 记录 `S2_asl_i_p_or`。
- **L791 EN**: Declares TableGen def record `S2_asl_i_p_xacc`.
  **L791 CN**: 声明 TableGen def 记录 `S2_asl_i_p_xacc`。
- **L792 EN**: Declares TableGen def record `S2_asl_i_r`.
  **L792 CN**: 声明 TableGen def 记录 `S2_asl_i_r`。
- **L793 EN**: Declares TableGen def record `S2_asl_i_r_acc`.
  **L793 CN**: 声明 TableGen def 记录 `S2_asl_i_r_acc`。
- **L794 EN**: Declares TableGen def record `S2_asl_i_r_and`.
  **L794 CN**: 声明 TableGen def 记录 `S2_asl_i_r_and`。
- **L795 EN**: Declares TableGen def record `S2_asl_i_r_nac`.
  **L795 CN**: 声明 TableGen def 记录 `S2_asl_i_r_nac`。
- **L796 EN**: Declares TableGen def record `S2_asl_i_r_or`.
  **L796 CN**: 声明 TableGen def 记录 `S2_asl_i_r_or`。
- **L797 EN**: Declares TableGen def record `S2_asl_i_r_sat`.
  **L797 CN**: 声明 TableGen def 记录 `S2_asl_i_r_sat`。
- **L798 EN**: Declares TableGen def record `S2_asl_i_r_xacc`.
  **L798 CN**: 声明 TableGen def 记录 `S2_asl_i_r_xacc`。
- **L799 EN**: Declares TableGen def record `S2_asl_i_vh`.
  **L799 CN**: 声明 TableGen def 记录 `S2_asl_i_vh`。
- **L800 EN**: Declares TableGen def record `S2_asl_i_vw`.
  **L800 CN**: 声明 TableGen def 记录 `S2_asl_i_vw`。
- **L801 EN**: Declares TableGen def record `S2_asl_r_p`.
  **L801 CN**: 声明 TableGen def 记录 `S2_asl_r_p`。
- **L802 EN**: Declares TableGen def record `S2_asl_r_p_acc`.
  **L802 CN**: 声明 TableGen def 记录 `S2_asl_r_p_acc`。
- **L803 EN**: Declares TableGen def record `S2_asl_r_p_and`.
  **L803 CN**: 声明 TableGen def 记录 `S2_asl_r_p_and`。
- **L804 EN**: Declares TableGen def record `S2_asl_r_p_nac`.
  **L804 CN**: 声明 TableGen def 记录 `S2_asl_r_p_nac`。
- **L805 EN**: Declares TableGen def record `S2_asl_r_p_or`.
  **L805 CN**: 声明 TableGen def 记录 `S2_asl_r_p_or`。
- **L806 EN**: Declares TableGen def record `S2_asl_r_p_xor`.
  **L806 CN**: 声明 TableGen def 记录 `S2_asl_r_p_xor`。
- **L807 EN**: Declares TableGen def record `S2_asl_r_r`.
  **L807 CN**: 声明 TableGen def 记录 `S2_asl_r_r`。
- **L808 EN**: Declares TableGen def record `S2_asl_r_r_acc`.
  **L808 CN**: 声明 TableGen def 记录 `S2_asl_r_r_acc`。
- **L809 EN**: Declares TableGen def record `S2_asl_r_r_and`.
  **L809 CN**: 声明 TableGen def 记录 `S2_asl_r_r_and`。
- **L810 EN**: Declares TableGen def record `S2_asl_r_r_nac`.
  **L810 CN**: 声明 TableGen def 记录 `S2_asl_r_r_nac`。
- **L811 EN**: Declares TableGen def record `S2_asl_r_r_or`.
  **L811 CN**: 声明 TableGen def 记录 `S2_asl_r_r_or`。
- **L812 EN**: Declares TableGen def record `S2_asl_r_r_sat`.
  **L812 CN**: 声明 TableGen def 记录 `S2_asl_r_r_sat`。

### Lines 813-840

````tablegen
def S2_asl_r_vh : HexagonBuiltin<"long long int(long long int, int)">;
def S2_asl_r_vw : HexagonBuiltin<"long long int(long long int, int)">;
def S2_asr_i_p : HexagonBuiltin<"long long int(long long int, unsigned _Constant int)">;
def S2_asr_i_p_acc : HexagonBuiltin<"long long int(long long int, long long int, unsigned _Constant int)">;
def S2_asr_i_p_and : HexagonBuiltin<"long long int(long long int, long long int, unsigned _Constant int)">;
def S2_asr_i_p_nac : HexagonBuiltin<"long long int(long long int, long long int, unsigned _Constant int)">;
def S2_asr_i_p_or : HexagonBuiltin<"long long int(long long int, long long int, unsigned _Constant int)">;
def S2_asr_i_p_rnd : HexagonBuiltin<"long long int(long long int, unsigned _Constant int)">;
def S2_asr_i_p_rnd_goodsyntax : HexagonBuiltin<"long long int(long long int, unsigned _Constant int)">;
def S2_asr_i_r : HexagonBuiltin<"int(int, unsigned _Constant int)">;
def S2_asr_i_r_acc : HexagonBuiltin<"int(int, int, unsigned _Constant int)">;
def S2_asr_i_r_and : HexagonBuiltin<"int(int, int, unsigned _Constant int)">;
def S2_asr_i_r_nac : HexagonBuiltin<"int(int, int, unsigned _Constant int)">;
def S2_asr_i_r_or : HexagonBuiltin<"int(int, int, unsigned _Constant int)">;
def S2_asr_i_r_rnd : HexagonBuiltin<"int(int, unsigned _Constant int)">;
def S2_asr_i_r_rnd_goodsyntax : HexagonBuiltin<"int(int, unsigned _Constant int)">;
def S2_asr_i_svw_trun : HexagonBuiltin<"int(long long int, unsigned _Constant int)">;
def S2_asr_i_vh : HexagonBuiltin<"long long int(long long int, unsigned _Constant int)">;
def S2_asr_i_vw : HexagonBuiltin<"long long int(long long int, unsigned _Constant int)">;
def S2_asr_r_p : HexagonBuiltin<"long long int(long long int, int)">;
def S2_asr_r_p_acc : HexagonBuiltin<"long long int(long long int, long long int, int)">;
def S2_asr_r_p_and : HexagonBuiltin<"long long int(long long int, long long int, int)">;
def S2_asr_r_p_nac : HexagonBuiltin<"long long int(long long int, long long int, int)">;
def S2_asr_r_p_or : HexagonBuiltin<"long long int(long long int, long long int, int)">;
def S2_asr_r_p_xor : HexagonBuiltin<"long long int(long long int, long long int, int)">;
def S2_asr_r_r : HexagonBuiltin<"int(int, int)">;
def S2_asr_r_r_acc : HexagonBuiltin<"int(int, int, int)">;
def S2_asr_r_r_and : HexagonBuiltin<"int(int, int, int)">;
````
- **L813 EN**: Declares TableGen def record `S2_asl_r_vh`.
  **L813 CN**: 声明 TableGen def 记录 `S2_asl_r_vh`。
- **L814 EN**: Declares TableGen def record `S2_asl_r_vw`.
  **L814 CN**: 声明 TableGen def 记录 `S2_asl_r_vw`。
- **L815 EN**: Declares TableGen def record `S2_asr_i_p`.
  **L815 CN**: 声明 TableGen def 记录 `S2_asr_i_p`。
- **L816 EN**: Declares TableGen def record `S2_asr_i_p_acc`.
  **L816 CN**: 声明 TableGen def 记录 `S2_asr_i_p_acc`。
- **L817 EN**: Declares TableGen def record `S2_asr_i_p_and`.
  **L817 CN**: 声明 TableGen def 记录 `S2_asr_i_p_and`。
- **L818 EN**: Declares TableGen def record `S2_asr_i_p_nac`.
  **L818 CN**: 声明 TableGen def 记录 `S2_asr_i_p_nac`。
- **L819 EN**: Declares TableGen def record `S2_asr_i_p_or`.
  **L819 CN**: 声明 TableGen def 记录 `S2_asr_i_p_or`。
- **L820 EN**: Declares TableGen def record `S2_asr_i_p_rnd`.
  **L820 CN**: 声明 TableGen def 记录 `S2_asr_i_p_rnd`。
- **L821 EN**: Declares TableGen def record `S2_asr_i_p_rnd_goodsyntax`.
  **L821 CN**: 声明 TableGen def 记录 `S2_asr_i_p_rnd_goodsyntax`。
- **L822 EN**: Declares TableGen def record `S2_asr_i_r`.
  **L822 CN**: 声明 TableGen def 记录 `S2_asr_i_r`。
- **L823 EN**: Declares TableGen def record `S2_asr_i_r_acc`.
  **L823 CN**: 声明 TableGen def 记录 `S2_asr_i_r_acc`。
- **L824 EN**: Declares TableGen def record `S2_asr_i_r_and`.
  **L824 CN**: 声明 TableGen def 记录 `S2_asr_i_r_and`。
- **L825 EN**: Declares TableGen def record `S2_asr_i_r_nac`.
  **L825 CN**: 声明 TableGen def 记录 `S2_asr_i_r_nac`。
- **L826 EN**: Declares TableGen def record `S2_asr_i_r_or`.
  **L826 CN**: 声明 TableGen def 记录 `S2_asr_i_r_or`。
- **L827 EN**: Declares TableGen def record `S2_asr_i_r_rnd`.
  **L827 CN**: 声明 TableGen def 记录 `S2_asr_i_r_rnd`。
- **L828 EN**: Declares TableGen def record `S2_asr_i_r_rnd_goodsyntax`.
  **L828 CN**: 声明 TableGen def 记录 `S2_asr_i_r_rnd_goodsyntax`。
- **L829 EN**: Declares TableGen def record `S2_asr_i_svw_trun`.
  **L829 CN**: 声明 TableGen def 记录 `S2_asr_i_svw_trun`。
- **L830 EN**: Declares TableGen def record `S2_asr_i_vh`.
  **L830 CN**: 声明 TableGen def 记录 `S2_asr_i_vh`。
- **L831 EN**: Declares TableGen def record `S2_asr_i_vw`.
  **L831 CN**: 声明 TableGen def 记录 `S2_asr_i_vw`。
- **L832 EN**: Declares TableGen def record `S2_asr_r_p`.
  **L832 CN**: 声明 TableGen def 记录 `S2_asr_r_p`。
- **L833 EN**: Declares TableGen def record `S2_asr_r_p_acc`.
  **L833 CN**: 声明 TableGen def 记录 `S2_asr_r_p_acc`。
- **L834 EN**: Declares TableGen def record `S2_asr_r_p_and`.
  **L834 CN**: 声明 TableGen def 记录 `S2_asr_r_p_and`。
- **L835 EN**: Declares TableGen def record `S2_asr_r_p_nac`.
  **L835 CN**: 声明 TableGen def 记录 `S2_asr_r_p_nac`。
- **L836 EN**: Declares TableGen def record `S2_asr_r_p_or`.
  **L836 CN**: 声明 TableGen def 记录 `S2_asr_r_p_or`。
- **L837 EN**: Declares TableGen def record `S2_asr_r_p_xor`.
  **L837 CN**: 声明 TableGen def 记录 `S2_asr_r_p_xor`。
- **L838 EN**: Declares TableGen def record `S2_asr_r_r`.
  **L838 CN**: 声明 TableGen def 记录 `S2_asr_r_r`。
- **L839 EN**: Declares TableGen def record `S2_asr_r_r_acc`.
  **L839 CN**: 声明 TableGen def 记录 `S2_asr_r_r_acc`。
- **L840 EN**: Declares TableGen def record `S2_asr_r_r_and`.
  **L840 CN**: 声明 TableGen def 记录 `S2_asr_r_r_and`。

### Lines 841-868

````tablegen
def S2_asr_r_r_nac : HexagonBuiltin<"int(int, int, int)">;
def S2_asr_r_r_or : HexagonBuiltin<"int(int, int, int)">;
def S2_asr_r_r_sat : HexagonBuiltin<"int(int, int)">;
def S2_asr_r_svw_trun : HexagonBuiltin<"int(long long int, int)">;
def S2_asr_r_vh : HexagonBuiltin<"long long int(long long int, int)">;
def S2_asr_r_vw : HexagonBuiltin<"long long int(long long int, int)">;
def S2_brev : HexagonBuiltin<"int(int)">;
def S2_brevp : HexagonBuiltin<"long long int(long long int)">;
def S2_cl0 : HexagonBuiltin<"int(int)">;
def S2_cl0p : HexagonBuiltin<"int(long long int)">;
def S2_cl1 : HexagonBuiltin<"int(int)">;
def S2_cl1p : HexagonBuiltin<"int(long long int)">;
def S2_clb : HexagonBuiltin<"int(int)">;
def S2_clbnorm : HexagonBuiltin<"int(int)">;
def S2_clbp : HexagonBuiltin<"int(long long int)">;
def S2_clrbit_i : HexagonBuiltin<"int(int, unsigned _Constant int)">;
def S2_clrbit_r : HexagonBuiltin<"int(int, int)">;
def S2_ct0 : HexagonBuiltin<"int(int)">;
def S2_ct0p : HexagonBuiltin<"int(long long int)">;
def S2_ct1 : HexagonBuiltin<"int(int)">;
def S2_ct1p : HexagonBuiltin<"int(long long int)">;
def S2_deinterleave : HexagonBuiltin<"long long int(long long int)">;
def S2_extractu : HexagonBuiltin<"int(int, unsigned _Constant int, unsigned _Constant int)">;
def S2_extractu_rp : HexagonBuiltin<"int(int, long long int)">;
def S2_extractup : HexagonBuiltin<"long long int(long long int, unsigned _Constant int, unsigned _Constant int)">;
def S2_extractup_rp : HexagonBuiltin<"long long int(long long int, long long int)">;
def S2_insert : HexagonBuiltin<"int(int, int, unsigned _Constant int, unsigned _Constant int)">;
def S2_insert_rp : HexagonBuiltin<"int(int, int, long long int)">;
````
- **L841 EN**: Declares TableGen def record `S2_asr_r_r_nac`.
  **L841 CN**: 声明 TableGen def 记录 `S2_asr_r_r_nac`。
- **L842 EN**: Declares TableGen def record `S2_asr_r_r_or`.
  **L842 CN**: 声明 TableGen def 记录 `S2_asr_r_r_or`。
- **L843 EN**: Declares TableGen def record `S2_asr_r_r_sat`.
  **L843 CN**: 声明 TableGen def 记录 `S2_asr_r_r_sat`。
- **L844 EN**: Declares TableGen def record `S2_asr_r_svw_trun`.
  **L844 CN**: 声明 TableGen def 记录 `S2_asr_r_svw_trun`。
- **L845 EN**: Declares TableGen def record `S2_asr_r_vh`.
  **L845 CN**: 声明 TableGen def 记录 `S2_asr_r_vh`。
- **L846 EN**: Declares TableGen def record `S2_asr_r_vw`.
  **L846 CN**: 声明 TableGen def 记录 `S2_asr_r_vw`。
- **L847 EN**: Declares TableGen def record `S2_brev`.
  **L847 CN**: 声明 TableGen def 记录 `S2_brev`。
- **L848 EN**: Declares TableGen def record `S2_brevp`.
  **L848 CN**: 声明 TableGen def 记录 `S2_brevp`。
- **L849 EN**: Declares TableGen def record `S2_cl0`.
  **L849 CN**: 声明 TableGen def 记录 `S2_cl0`。
- **L850 EN**: Declares TableGen def record `S2_cl0p`.
  **L850 CN**: 声明 TableGen def 记录 `S2_cl0p`。
- **L851 EN**: Declares TableGen def record `S2_cl1`.
  **L851 CN**: 声明 TableGen def 记录 `S2_cl1`。
- **L852 EN**: Declares TableGen def record `S2_cl1p`.
  **L852 CN**: 声明 TableGen def 记录 `S2_cl1p`。
- **L853 EN**: Declares TableGen def record `S2_clb`.
  **L853 CN**: 声明 TableGen def 记录 `S2_clb`。
- **L854 EN**: Declares TableGen def record `S2_clbnorm`.
  **L854 CN**: 声明 TableGen def 记录 `S2_clbnorm`。
- **L855 EN**: Declares TableGen def record `S2_clbp`.
  **L855 CN**: 声明 TableGen def 记录 `S2_clbp`。
- **L856 EN**: Declares TableGen def record `S2_clrbit_i`.
  **L856 CN**: 声明 TableGen def 记录 `S2_clrbit_i`。
- **L857 EN**: Declares TableGen def record `S2_clrbit_r`.
  **L857 CN**: 声明 TableGen def 记录 `S2_clrbit_r`。
- **L858 EN**: Declares TableGen def record `S2_ct0`.
  **L858 CN**: 声明 TableGen def 记录 `S2_ct0`。
- **L859 EN**: Declares TableGen def record `S2_ct0p`.
  **L859 CN**: 声明 TableGen def 记录 `S2_ct0p`。
- **L860 EN**: Declares TableGen def record `S2_ct1`.
  **L860 CN**: 声明 TableGen def 记录 `S2_ct1`。
- **L861 EN**: Declares TableGen def record `S2_ct1p`.
  **L861 CN**: 声明 TableGen def 记录 `S2_ct1p`。
- **L862 EN**: Declares TableGen def record `S2_deinterleave`.
  **L862 CN**: 声明 TableGen def 记录 `S2_deinterleave`。
- **L863 EN**: Declares TableGen def record `S2_extractu`.
  **L863 CN**: 声明 TableGen def 记录 `S2_extractu`。
- **L864 EN**: Declares TableGen def record `S2_extractu_rp`.
  **L864 CN**: 声明 TableGen def 记录 `S2_extractu_rp`。
- **L865 EN**: Declares TableGen def record `S2_extractup`.
  **L865 CN**: 声明 TableGen def 记录 `S2_extractup`。
- **L866 EN**: Declares TableGen def record `S2_extractup_rp`.
  **L866 CN**: 声明 TableGen def 记录 `S2_extractup_rp`。
- **L867 EN**: Declares TableGen def record `S2_insert`.
  **L867 CN**: 声明 TableGen def 记录 `S2_insert`。
- **L868 EN**: Declares TableGen def record `S2_insert_rp`.
  **L868 CN**: 声明 TableGen def 记录 `S2_insert_rp`。

### Lines 869-896

````tablegen
def S2_insertp : HexagonBuiltin<"long long int(long long int, long long int, unsigned _Constant int, unsigned _Constant int)">;
def S2_insertp_rp : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
def S2_interleave : HexagonBuiltin<"long long int(long long int)">;
def S2_lfsp : HexagonBuiltin<"long long int(long long int, long long int)">;
def S2_lsl_r_p : HexagonBuiltin<"long long int(long long int, int)">;
def S2_lsl_r_p_acc : HexagonBuiltin<"long long int(long long int, long long int, int)">;
def S2_lsl_r_p_and : HexagonBuiltin<"long long int(long long int, long long int, int)">;
def S2_lsl_r_p_nac : HexagonBuiltin<"long long int(long long int, long long int, int)">;
def S2_lsl_r_p_or : HexagonBuiltin<"long long int(long long int, long long int, int)">;
def S2_lsl_r_p_xor : HexagonBuiltin<"long long int(long long int, long long int, int)">;
def S2_lsl_r_r : HexagonBuiltin<"int(int, int)">;
def S2_lsl_r_r_acc : HexagonBuiltin<"int(int, int, int)">;
def S2_lsl_r_r_and : HexagonBuiltin<"int(int, int, int)">;
def S2_lsl_r_r_nac : HexagonBuiltin<"int(int, int, int)">;
def S2_lsl_r_r_or : HexagonBuiltin<"int(int, int, int)">;
def S2_lsl_r_vh : HexagonBuiltin<"long long int(long long int, int)">;
def S2_lsl_r_vw : HexagonBuiltin<"long long int(long long int, int)">;
def S2_lsr_i_p : HexagonBuiltin<"long long int(long long int, unsigned _Constant int)">;
def S2_lsr_i_p_acc : HexagonBuiltin<"long long int(long long int, long long int, unsigned _Constant int)">;
def S2_lsr_i_p_and : HexagonBuiltin<"long long int(long long int, long long int, unsigned _Constant int)">;
def S2_lsr_i_p_nac : HexagonBuiltin<"long long int(long long int, long long int, unsigned _Constant int)">;
def S2_lsr_i_p_or : HexagonBuiltin<"long long int(long long int, long long int, unsigned _Constant int)">;
def S2_lsr_i_p_xacc : HexagonBuiltin<"long long int(long long int, long long int, unsigned _Constant int)">;
def S2_lsr_i_r : HexagonBuiltin<"int(int, unsigned _Constant int)">;
def S2_lsr_i_r_acc : HexagonBuiltin<"int(int, int, unsigned _Constant int)">;
def S2_lsr_i_r_and : HexagonBuiltin<"int(int, int, unsigned _Constant int)">;
def S2_lsr_i_r_nac : HexagonBuiltin<"int(int, int, unsigned _Constant int)">;
def S2_lsr_i_r_or : HexagonBuiltin<"int(int, int, unsigned _Constant int)">;
````
- **L869 EN**: Declares TableGen def record `S2_insertp`.
  **L869 CN**: 声明 TableGen def 记录 `S2_insertp`。
- **L870 EN**: Declares TableGen def record `S2_insertp_rp`.
  **L870 CN**: 声明 TableGen def 记录 `S2_insertp_rp`。
- **L871 EN**: Declares TableGen def record `S2_interleave`.
  **L871 CN**: 声明 TableGen def 记录 `S2_interleave`。
- **L872 EN**: Declares TableGen def record `S2_lfsp`.
  **L872 CN**: 声明 TableGen def 记录 `S2_lfsp`。
- **L873 EN**: Declares TableGen def record `S2_lsl_r_p`.
  **L873 CN**: 声明 TableGen def 记录 `S2_lsl_r_p`。
- **L874 EN**: Declares TableGen def record `S2_lsl_r_p_acc`.
  **L874 CN**: 声明 TableGen def 记录 `S2_lsl_r_p_acc`。
- **L875 EN**: Declares TableGen def record `S2_lsl_r_p_and`.
  **L875 CN**: 声明 TableGen def 记录 `S2_lsl_r_p_and`。
- **L876 EN**: Declares TableGen def record `S2_lsl_r_p_nac`.
  **L876 CN**: 声明 TableGen def 记录 `S2_lsl_r_p_nac`。
- **L877 EN**: Declares TableGen def record `S2_lsl_r_p_or`.
  **L877 CN**: 声明 TableGen def 记录 `S2_lsl_r_p_or`。
- **L878 EN**: Declares TableGen def record `S2_lsl_r_p_xor`.
  **L878 CN**: 声明 TableGen def 记录 `S2_lsl_r_p_xor`。
- **L879 EN**: Declares TableGen def record `S2_lsl_r_r`.
  **L879 CN**: 声明 TableGen def 记录 `S2_lsl_r_r`。
- **L880 EN**: Declares TableGen def record `S2_lsl_r_r_acc`.
  **L880 CN**: 声明 TableGen def 记录 `S2_lsl_r_r_acc`。
- **L881 EN**: Declares TableGen def record `S2_lsl_r_r_and`.
  **L881 CN**: 声明 TableGen def 记录 `S2_lsl_r_r_and`。
- **L882 EN**: Declares TableGen def record `S2_lsl_r_r_nac`.
  **L882 CN**: 声明 TableGen def 记录 `S2_lsl_r_r_nac`。
- **L883 EN**: Declares TableGen def record `S2_lsl_r_r_or`.
  **L883 CN**: 声明 TableGen def 记录 `S2_lsl_r_r_or`。
- **L884 EN**: Declares TableGen def record `S2_lsl_r_vh`.
  **L884 CN**: 声明 TableGen def 记录 `S2_lsl_r_vh`。
- **L885 EN**: Declares TableGen def record `S2_lsl_r_vw`.
  **L885 CN**: 声明 TableGen def 记录 `S2_lsl_r_vw`。
- **L886 EN**: Declares TableGen def record `S2_lsr_i_p`.
  **L886 CN**: 声明 TableGen def 记录 `S2_lsr_i_p`。
- **L887 EN**: Declares TableGen def record `S2_lsr_i_p_acc`.
  **L887 CN**: 声明 TableGen def 记录 `S2_lsr_i_p_acc`。
- **L888 EN**: Declares TableGen def record `S2_lsr_i_p_and`.
  **L888 CN**: 声明 TableGen def 记录 `S2_lsr_i_p_and`。
- **L889 EN**: Declares TableGen def record `S2_lsr_i_p_nac`.
  **L889 CN**: 声明 TableGen def 记录 `S2_lsr_i_p_nac`。
- **L890 EN**: Declares TableGen def record `S2_lsr_i_p_or`.
  **L890 CN**: 声明 TableGen def 记录 `S2_lsr_i_p_or`。
- **L891 EN**: Declares TableGen def record `S2_lsr_i_p_xacc`.
  **L891 CN**: 声明 TableGen def 记录 `S2_lsr_i_p_xacc`。
- **L892 EN**: Declares TableGen def record `S2_lsr_i_r`.
  **L892 CN**: 声明 TableGen def 记录 `S2_lsr_i_r`。
- **L893 EN**: Declares TableGen def record `S2_lsr_i_r_acc`.
  **L893 CN**: 声明 TableGen def 记录 `S2_lsr_i_r_acc`。
- **L894 EN**: Declares TableGen def record `S2_lsr_i_r_and`.
  **L894 CN**: 声明 TableGen def 记录 `S2_lsr_i_r_and`。
- **L895 EN**: Declares TableGen def record `S2_lsr_i_r_nac`.
  **L895 CN**: 声明 TableGen def 记录 `S2_lsr_i_r_nac`。
- **L896 EN**: Declares TableGen def record `S2_lsr_i_r_or`.
  **L896 CN**: 声明 TableGen def 记录 `S2_lsr_i_r_or`。

### Lines 897-924

````tablegen
def S2_lsr_i_r_xacc : HexagonBuiltin<"int(int, int, unsigned _Constant int)">;
def S2_lsr_i_vh : HexagonBuiltin<"long long int(long long int, unsigned _Constant int)">;
def S2_lsr_i_vw : HexagonBuiltin<"long long int(long long int, unsigned _Constant int)">;
def S2_lsr_r_p : HexagonBuiltin<"long long int(long long int, int)">;
def S2_lsr_r_p_acc : HexagonBuiltin<"long long int(long long int, long long int, int)">;
def S2_lsr_r_p_and : HexagonBuiltin<"long long int(long long int, long long int, int)">;
def S2_lsr_r_p_nac : HexagonBuiltin<"long long int(long long int, long long int, int)">;
def S2_lsr_r_p_or : HexagonBuiltin<"long long int(long long int, long long int, int)">;
def S2_lsr_r_p_xor : HexagonBuiltin<"long long int(long long int, long long int, int)">;
def S2_lsr_r_r : HexagonBuiltin<"int(int, int)">;
def S2_lsr_r_r_acc : HexagonBuiltin<"int(int, int, int)">;
def S2_lsr_r_r_and : HexagonBuiltin<"int(int, int, int)">;
def S2_lsr_r_r_nac : HexagonBuiltin<"int(int, int, int)">;
def S2_lsr_r_r_or : HexagonBuiltin<"int(int, int, int)">;
def S2_lsr_r_vh : HexagonBuiltin<"long long int(long long int, int)">;
def S2_lsr_r_vw : HexagonBuiltin<"long long int(long long int, int)">;
def S2_packhl : HexagonBuiltin<"long long int(int, int)">;
def S2_parityp : HexagonBuiltin<"int(long long int, long long int)">;
def S2_setbit_i : HexagonBuiltin<"int(int, unsigned _Constant int)">;
def S2_setbit_r : HexagonBuiltin<"int(int, int)">;
def S2_shuffeb : HexagonBuiltin<"long long int(long long int, long long int)">;
def S2_shuffeh : HexagonBuiltin<"long long int(long long int, long long int)">;
def S2_shuffob : HexagonBuiltin<"long long int(long long int, long long int)">;
def S2_shuffoh : HexagonBuiltin<"long long int(long long int, long long int)">;
def S2_svsathb : HexagonBuiltin<"int(int)">;
def S2_svsathub : HexagonBuiltin<"int(int)">;
def S2_tableidxb_goodsyntax : HexagonBuiltin<"int(int, int, unsigned _Constant int, unsigned _Constant int)">;
def S2_tableidxd_goodsyntax : HexagonBuiltin<"int(int, int, unsigned _Constant int, unsigned _Constant int)">;
````
- **L897 EN**: Declares TableGen def record `S2_lsr_i_r_xacc`.
  **L897 CN**: 声明 TableGen def 记录 `S2_lsr_i_r_xacc`。
- **L898 EN**: Declares TableGen def record `S2_lsr_i_vh`.
  **L898 CN**: 声明 TableGen def 记录 `S2_lsr_i_vh`。
- **L899 EN**: Declares TableGen def record `S2_lsr_i_vw`.
  **L899 CN**: 声明 TableGen def 记录 `S2_lsr_i_vw`。
- **L900 EN**: Declares TableGen def record `S2_lsr_r_p`.
  **L900 CN**: 声明 TableGen def 记录 `S2_lsr_r_p`。
- **L901 EN**: Declares TableGen def record `S2_lsr_r_p_acc`.
  **L901 CN**: 声明 TableGen def 记录 `S2_lsr_r_p_acc`。
- **L902 EN**: Declares TableGen def record `S2_lsr_r_p_and`.
  **L902 CN**: 声明 TableGen def 记录 `S2_lsr_r_p_and`。
- **L903 EN**: Declares TableGen def record `S2_lsr_r_p_nac`.
  **L903 CN**: 声明 TableGen def 记录 `S2_lsr_r_p_nac`。
- **L904 EN**: Declares TableGen def record `S2_lsr_r_p_or`.
  **L904 CN**: 声明 TableGen def 记录 `S2_lsr_r_p_or`。
- **L905 EN**: Declares TableGen def record `S2_lsr_r_p_xor`.
  **L905 CN**: 声明 TableGen def 记录 `S2_lsr_r_p_xor`。
- **L906 EN**: Declares TableGen def record `S2_lsr_r_r`.
  **L906 CN**: 声明 TableGen def 记录 `S2_lsr_r_r`。
- **L907 EN**: Declares TableGen def record `S2_lsr_r_r_acc`.
  **L907 CN**: 声明 TableGen def 记录 `S2_lsr_r_r_acc`。
- **L908 EN**: Declares TableGen def record `S2_lsr_r_r_and`.
  **L908 CN**: 声明 TableGen def 记录 `S2_lsr_r_r_and`。
- **L909 EN**: Declares TableGen def record `S2_lsr_r_r_nac`.
  **L909 CN**: 声明 TableGen def 记录 `S2_lsr_r_r_nac`。
- **L910 EN**: Declares TableGen def record `S2_lsr_r_r_or`.
  **L910 CN**: 声明 TableGen def 记录 `S2_lsr_r_r_or`。
- **L911 EN**: Declares TableGen def record `S2_lsr_r_vh`.
  **L911 CN**: 声明 TableGen def 记录 `S2_lsr_r_vh`。
- **L912 EN**: Declares TableGen def record `S2_lsr_r_vw`.
  **L912 CN**: 声明 TableGen def 记录 `S2_lsr_r_vw`。
- **L913 EN**: Declares TableGen def record `S2_packhl`.
  **L913 CN**: 声明 TableGen def 记录 `S2_packhl`。
- **L914 EN**: Declares TableGen def record `S2_parityp`.
  **L914 CN**: 声明 TableGen def 记录 `S2_parityp`。
- **L915 EN**: Declares TableGen def record `S2_setbit_i`.
  **L915 CN**: 声明 TableGen def 记录 `S2_setbit_i`。
- **L916 EN**: Declares TableGen def record `S2_setbit_r`.
  **L916 CN**: 声明 TableGen def 记录 `S2_setbit_r`。
- **L917 EN**: Declares TableGen def record `S2_shuffeb`.
  **L917 CN**: 声明 TableGen def 记录 `S2_shuffeb`。
- **L918 EN**: Declares TableGen def record `S2_shuffeh`.
  **L918 CN**: 声明 TableGen def 记录 `S2_shuffeh`。
- **L919 EN**: Declares TableGen def record `S2_shuffob`.
  **L919 CN**: 声明 TableGen def 记录 `S2_shuffob`。
- **L920 EN**: Declares TableGen def record `S2_shuffoh`.
  **L920 CN**: 声明 TableGen def 记录 `S2_shuffoh`。
- **L921 EN**: Declares TableGen def record `S2_svsathb`.
  **L921 CN**: 声明 TableGen def 记录 `S2_svsathb`。
- **L922 EN**: Declares TableGen def record `S2_svsathub`.
  **L922 CN**: 声明 TableGen def 记录 `S2_svsathub`。
- **L923 EN**: Declares TableGen def record `S2_tableidxb_goodsyntax`.
  **L923 CN**: 声明 TableGen def 记录 `S2_tableidxb_goodsyntax`。
- **L924 EN**: Declares TableGen def record `S2_tableidxd_goodsyntax`.
  **L924 CN**: 声明 TableGen def 记录 `S2_tableidxd_goodsyntax`。

### Lines 925-952

````tablegen
def S2_tableidxh_goodsyntax : HexagonBuiltin<"int(int, int, unsigned _Constant int, unsigned _Constant int)">;
def S2_tableidxw_goodsyntax : HexagonBuiltin<"int(int, int, unsigned _Constant int, unsigned _Constant int)">;
def S2_togglebit_i : HexagonBuiltin<"int(int, unsigned _Constant int)">;
def S2_togglebit_r : HexagonBuiltin<"int(int, int)">;
def S2_tstbit_i : HexagonBuiltin<"int(int, unsigned _Constant int)">;
def S2_tstbit_r : HexagonBuiltin<"int(int, int)">;
def S2_valignib : HexagonBuiltin<"long long int(long long int, long long int, unsigned _Constant int)">;
def S2_valignrb : HexagonBuiltin<"long long int(long long int, long long int, int)">;
def S2_vcnegh : HexagonBuiltin<"long long int(long long int, int)">;
def S2_vcrotate : HexagonBuiltin<"long long int(long long int, int)">;
def S2_vrcnegh : HexagonBuiltin<"long long int(long long int, long long int, int)">;
def S2_vrndpackwh : HexagonBuiltin<"int(long long int)">;
def S2_vrndpackwhs : HexagonBuiltin<"int(long long int)">;
def S2_vsathb : HexagonBuiltin<"int(long long int)">;
def S2_vsathb_nopack : HexagonBuiltin<"long long int(long long int)">;
def S2_vsathub : HexagonBuiltin<"int(long long int)">;
def S2_vsathub_nopack : HexagonBuiltin<"long long int(long long int)">;
def S2_vsatwh : HexagonBuiltin<"int(long long int)">;
def S2_vsatwh_nopack : HexagonBuiltin<"long long int(long long int)">;
def S2_vsatwuh : HexagonBuiltin<"int(long long int)">;
def S2_vsatwuh_nopack : HexagonBuiltin<"long long int(long long int)">;
def S2_vsplatrb : HexagonBuiltin<"int(int)">;
def S2_vsplatrh : HexagonBuiltin<"long long int(int)">;
def S2_vspliceib : HexagonBuiltin<"long long int(long long int, long long int, unsigned _Constant int)">;
def S2_vsplicerb : HexagonBuiltin<"long long int(long long int, long long int, int)">;
def S2_vsxtbh : HexagonBuiltin<"long long int(int)">;
def S2_vsxthw : HexagonBuiltin<"long long int(int)">;
def S2_vtrunehb : HexagonBuiltin<"int(long long int)">;
````
- **L925 EN**: Declares TableGen def record `S2_tableidxh_goodsyntax`.
  **L925 CN**: 声明 TableGen def 记录 `S2_tableidxh_goodsyntax`。
- **L926 EN**: Declares TableGen def record `S2_tableidxw_goodsyntax`.
  **L926 CN**: 声明 TableGen def 记录 `S2_tableidxw_goodsyntax`。
- **L927 EN**: Declares TableGen def record `S2_togglebit_i`.
  **L927 CN**: 声明 TableGen def 记录 `S2_togglebit_i`。
- **L928 EN**: Declares TableGen def record `S2_togglebit_r`.
  **L928 CN**: 声明 TableGen def 记录 `S2_togglebit_r`。
- **L929 EN**: Declares TableGen def record `S2_tstbit_i`.
  **L929 CN**: 声明 TableGen def 记录 `S2_tstbit_i`。
- **L930 EN**: Declares TableGen def record `S2_tstbit_r`.
  **L930 CN**: 声明 TableGen def 记录 `S2_tstbit_r`。
- **L931 EN**: Declares TableGen def record `S2_valignib`.
  **L931 CN**: 声明 TableGen def 记录 `S2_valignib`。
- **L932 EN**: Declares TableGen def record `S2_valignrb`.
  **L932 CN**: 声明 TableGen def 记录 `S2_valignrb`。
- **L933 EN**: Declares TableGen def record `S2_vcnegh`.
  **L933 CN**: 声明 TableGen def 记录 `S2_vcnegh`。
- **L934 EN**: Declares TableGen def record `S2_vcrotate`.
  **L934 CN**: 声明 TableGen def 记录 `S2_vcrotate`。
- **L935 EN**: Declares TableGen def record `S2_vrcnegh`.
  **L935 CN**: 声明 TableGen def 记录 `S2_vrcnegh`。
- **L936 EN**: Declares TableGen def record `S2_vrndpackwh`.
  **L936 CN**: 声明 TableGen def 记录 `S2_vrndpackwh`。
- **L937 EN**: Declares TableGen def record `S2_vrndpackwhs`.
  **L937 CN**: 声明 TableGen def 记录 `S2_vrndpackwhs`。
- **L938 EN**: Declares TableGen def record `S2_vsathb`.
  **L938 CN**: 声明 TableGen def 记录 `S2_vsathb`。
- **L939 EN**: Declares TableGen def record `S2_vsathb_nopack`.
  **L939 CN**: 声明 TableGen def 记录 `S2_vsathb_nopack`。
- **L940 EN**: Declares TableGen def record `S2_vsathub`.
  **L940 CN**: 声明 TableGen def 记录 `S2_vsathub`。
- **L941 EN**: Declares TableGen def record `S2_vsathub_nopack`.
  **L941 CN**: 声明 TableGen def 记录 `S2_vsathub_nopack`。
- **L942 EN**: Declares TableGen def record `S2_vsatwh`.
  **L942 CN**: 声明 TableGen def 记录 `S2_vsatwh`。
- **L943 EN**: Declares TableGen def record `S2_vsatwh_nopack`.
  **L943 CN**: 声明 TableGen def 记录 `S2_vsatwh_nopack`。
- **L944 EN**: Declares TableGen def record `S2_vsatwuh`.
  **L944 CN**: 声明 TableGen def 记录 `S2_vsatwuh`。
- **L945 EN**: Declares TableGen def record `S2_vsatwuh_nopack`.
  **L945 CN**: 声明 TableGen def 记录 `S2_vsatwuh_nopack`。
- **L946 EN**: Declares TableGen def record `S2_vsplatrb`.
  **L946 CN**: 声明 TableGen def 记录 `S2_vsplatrb`。
- **L947 EN**: Declares TableGen def record `S2_vsplatrh`.
  **L947 CN**: 声明 TableGen def 记录 `S2_vsplatrh`。
- **L948 EN**: Declares TableGen def record `S2_vspliceib`.
  **L948 CN**: 声明 TableGen def 记录 `S2_vspliceib`。
- **L949 EN**: Declares TableGen def record `S2_vsplicerb`.
  **L949 CN**: 声明 TableGen def 记录 `S2_vsplicerb`。
- **L950 EN**: Declares TableGen def record `S2_vsxtbh`.
  **L950 CN**: 声明 TableGen def 记录 `S2_vsxtbh`。
- **L951 EN**: Declares TableGen def record `S2_vsxthw`.
  **L951 CN**: 声明 TableGen def 记录 `S2_vsxthw`。
- **L952 EN**: Declares TableGen def record `S2_vtrunehb`.
  **L952 CN**: 声明 TableGen def 记录 `S2_vtrunehb`。

### Lines 953-980

````tablegen
def S2_vtrunewh : HexagonBuiltin<"long long int(long long int, long long int)">;
def S2_vtrunohb : HexagonBuiltin<"int(long long int)">;
def S2_vtrunowh : HexagonBuiltin<"long long int(long long int, long long int)">;
def S2_vzxtbh : HexagonBuiltin<"long long int(int)">;
def S2_vzxthw : HexagonBuiltin<"long long int(int)">;
def S4_addaddi : HexagonBuiltin<"int(int, int, _Constant int)">;
def S4_addi_asl_ri : HexagonBuiltin<"int(unsigned _Constant int, int, unsigned _Constant int)">;
def S4_addi_lsr_ri : HexagonBuiltin<"int(unsigned _Constant int, int, unsigned _Constant int)">;
def S4_andi_asl_ri : HexagonBuiltin<"int(unsigned _Constant int, int, unsigned _Constant int)">;
def S4_andi_lsr_ri : HexagonBuiltin<"int(unsigned _Constant int, int, unsigned _Constant int)">;
def S4_clbaddi : HexagonBuiltin<"int(int, _Constant int)">;
def S4_clbpaddi : HexagonBuiltin<"int(long long int, _Constant int)">;
def S4_clbpnorm : HexagonBuiltin<"int(long long int)">;
def S4_extract : HexagonBuiltin<"int(int, unsigned _Constant int, unsigned _Constant int)">;
def S4_extract_rp : HexagonBuiltin<"int(int, long long int)">;
def S4_extractp : HexagonBuiltin<"long long int(long long int, unsigned _Constant int, unsigned _Constant int)">;
def S4_extractp_rp : HexagonBuiltin<"long long int(long long int, long long int)">;
def S4_lsli : HexagonBuiltin<"int(_Constant int, int)">;
def S4_ntstbit_i : HexagonBuiltin<"int(int, unsigned _Constant int)">;
def S4_ntstbit_r : HexagonBuiltin<"int(int, int)">;
def S4_or_andi : HexagonBuiltin<"int(int, int, _Constant int)">;
def S4_or_andix : HexagonBuiltin<"int(int, int, _Constant int)">;
def S4_or_ori : HexagonBuiltin<"int(int, int, _Constant int)">;
def S4_ori_asl_ri : HexagonBuiltin<"int(unsigned _Constant int, int, unsigned _Constant int)">;
def S4_ori_lsr_ri : HexagonBuiltin<"int(unsigned _Constant int, int, unsigned _Constant int)">;
def S4_parity : HexagonBuiltin<"int(int, int)">;
def S4_subaddi : HexagonBuiltin<"int(int, _Constant int, int)">;
def S4_subi_asl_ri : HexagonBuiltin<"int(unsigned _Constant int, int, unsigned _Constant int)">;
````
- **L953 EN**: Declares TableGen def record `S2_vtrunewh`.
  **L953 CN**: 声明 TableGen def 记录 `S2_vtrunewh`。
- **L954 EN**: Declares TableGen def record `S2_vtrunohb`.
  **L954 CN**: 声明 TableGen def 记录 `S2_vtrunohb`。
- **L955 EN**: Declares TableGen def record `S2_vtrunowh`.
  **L955 CN**: 声明 TableGen def 记录 `S2_vtrunowh`。
- **L956 EN**: Declares TableGen def record `S2_vzxtbh`.
  **L956 CN**: 声明 TableGen def 记录 `S2_vzxtbh`。
- **L957 EN**: Declares TableGen def record `S2_vzxthw`.
  **L957 CN**: 声明 TableGen def 记录 `S2_vzxthw`。
- **L958 EN**: Declares TableGen def record `S4_addaddi`.
  **L958 CN**: 声明 TableGen def 记录 `S4_addaddi`。
- **L959 EN**: Declares TableGen def record `S4_addi_asl_ri`.
  **L959 CN**: 声明 TableGen def 记录 `S4_addi_asl_ri`。
- **L960 EN**: Declares TableGen def record `S4_addi_lsr_ri`.
  **L960 CN**: 声明 TableGen def 记录 `S4_addi_lsr_ri`。
- **L961 EN**: Declares TableGen def record `S4_andi_asl_ri`.
  **L961 CN**: 声明 TableGen def 记录 `S4_andi_asl_ri`。
- **L962 EN**: Declares TableGen def record `S4_andi_lsr_ri`.
  **L962 CN**: 声明 TableGen def 记录 `S4_andi_lsr_ri`。
- **L963 EN**: Declares TableGen def record `S4_clbaddi`.
  **L963 CN**: 声明 TableGen def 记录 `S4_clbaddi`。
- **L964 EN**: Declares TableGen def record `S4_clbpaddi`.
  **L964 CN**: 声明 TableGen def 记录 `S4_clbpaddi`。
- **L965 EN**: Declares TableGen def record `S4_clbpnorm`.
  **L965 CN**: 声明 TableGen def 记录 `S4_clbpnorm`。
- **L966 EN**: Declares TableGen def record `S4_extract`.
  **L966 CN**: 声明 TableGen def 记录 `S4_extract`。
- **L967 EN**: Declares TableGen def record `S4_extract_rp`.
  **L967 CN**: 声明 TableGen def 记录 `S4_extract_rp`。
- **L968 EN**: Declares TableGen def record `S4_extractp`.
  **L968 CN**: 声明 TableGen def 记录 `S4_extractp`。
- **L969 EN**: Declares TableGen def record `S4_extractp_rp`.
  **L969 CN**: 声明 TableGen def 记录 `S4_extractp_rp`。
- **L970 EN**: Declares TableGen def record `S4_lsli`.
  **L970 CN**: 声明 TableGen def 记录 `S4_lsli`。
- **L971 EN**: Declares TableGen def record `S4_ntstbit_i`.
  **L971 CN**: 声明 TableGen def 记录 `S4_ntstbit_i`。
- **L972 EN**: Declares TableGen def record `S4_ntstbit_r`.
  **L972 CN**: 声明 TableGen def 记录 `S4_ntstbit_r`。
- **L973 EN**: Declares TableGen def record `S4_or_andi`.
  **L973 CN**: 声明 TableGen def 记录 `S4_or_andi`。
- **L974 EN**: Declares TableGen def record `S4_or_andix`.
  **L974 CN**: 声明 TableGen def 记录 `S4_or_andix`。
- **L975 EN**: Declares TableGen def record `S4_or_ori`.
  **L975 CN**: 声明 TableGen def 记录 `S4_or_ori`。
- **L976 EN**: Declares TableGen def record `S4_ori_asl_ri`.
  **L976 CN**: 声明 TableGen def 记录 `S4_ori_asl_ri`。
- **L977 EN**: Declares TableGen def record `S4_ori_lsr_ri`.
  **L977 CN**: 声明 TableGen def 记录 `S4_ori_lsr_ri`。
- **L978 EN**: Declares TableGen def record `S4_parity`.
  **L978 CN**: 声明 TableGen def 记录 `S4_parity`。
- **L979 EN**: Declares TableGen def record `S4_subaddi`.
  **L979 CN**: 声明 TableGen def 记录 `S4_subaddi`。
- **L980 EN**: Declares TableGen def record `S4_subi_asl_ri`.
  **L980 CN**: 声明 TableGen def 记录 `S4_subi_asl_ri`。

### Lines 981-1008

````tablegen
def S4_subi_lsr_ri : HexagonBuiltin<"int(unsigned _Constant int, int, unsigned _Constant int)">;
def S4_vrcrotate : HexagonBuiltin<"long long int(long long int, int, unsigned _Constant int)">;
def S4_vrcrotate_acc : HexagonBuiltin<"long long int(long long int, long long int, int, unsigned _Constant int)">;
def S4_vxaddsubh : HexagonBuiltin<"long long int(long long int, long long int)">;
def S4_vxaddsubhr : HexagonBuiltin<"long long int(long long int, long long int)">;
def S4_vxaddsubw : HexagonBuiltin<"long long int(long long int, long long int)">;
def S4_vxsubaddh : HexagonBuiltin<"long long int(long long int, long long int)">;
def S4_vxsubaddhr : HexagonBuiltin<"long long int(long long int, long long int)">;
def S4_vxsubaddw : HexagonBuiltin<"long long int(long long int, long long int)">;
def S5_asrhub_rnd_sat_goodsyntax : HexagonBuiltin<"int(long long int, unsigned _Constant int)">;
def S5_asrhub_sat : HexagonBuiltin<"int(long long int, unsigned _Constant int)">;
def S5_popcountp : HexagonBuiltin<"int(long long int)">;
def S5_vasrhrnd_goodsyntax : HexagonBuiltin<"long long int(long long int, unsigned _Constant int)">;
def Y2_dccleana : HexagonBuiltin<"void(void *)">;
def Y2_dccleaninva : HexagonBuiltin<"void(void *)">;
def Y2_dcfetch : HexagonBuiltin<"void(void *)">;
def Y2_dcinva : HexagonBuiltin<"void(void *)">;
def Y2_dczeroa : HexagonBuiltin<"void(void *)">;
def Y4_l2fetch : HexagonBuiltin<"void(void *, int)">;
def Y5_l2fetch : HexagonBuiltin<"void(void *, long long int)">;

// V60 Scalar Instructions.

let Features = V60.Features in {
  def S6_rol_i_p : HexagonBuiltin<"long long int(long long int, unsigned _Constant int)">;
  def S6_rol_i_p_acc : HexagonBuiltin<"long long int(long long int, long long int, unsigned _Constant int)">;
  def S6_rol_i_p_and : HexagonBuiltin<"long long int(long long int, long long int, unsigned _Constant int)">;
  def S6_rol_i_p_nac : HexagonBuiltin<"long long int(long long int, long long int, unsigned _Constant int)">;
````
- **L981 EN**: Declares TableGen def record `S4_subi_lsr_ri`.
  **L981 CN**: 声明 TableGen def 记录 `S4_subi_lsr_ri`。
- **L982 EN**: Declares TableGen def record `S4_vrcrotate`.
  **L982 CN**: 声明 TableGen def 记录 `S4_vrcrotate`。
- **L983 EN**: Declares TableGen def record `S4_vrcrotate_acc`.
  **L983 CN**: 声明 TableGen def 记录 `S4_vrcrotate_acc`。
- **L984 EN**: Declares TableGen def record `S4_vxaddsubh`.
  **L984 CN**: 声明 TableGen def 记录 `S4_vxaddsubh`。
- **L985 EN**: Declares TableGen def record `S4_vxaddsubhr`.
  **L985 CN**: 声明 TableGen def 记录 `S4_vxaddsubhr`。
- **L986 EN**: Declares TableGen def record `S4_vxaddsubw`.
  **L986 CN**: 声明 TableGen def 记录 `S4_vxaddsubw`。
- **L987 EN**: Declares TableGen def record `S4_vxsubaddh`.
  **L987 CN**: 声明 TableGen def 记录 `S4_vxsubaddh`。
- **L988 EN**: Declares TableGen def record `S4_vxsubaddhr`.
  **L988 CN**: 声明 TableGen def 记录 `S4_vxsubaddhr`。
- **L989 EN**: Declares TableGen def record `S4_vxsubaddw`.
  **L989 CN**: 声明 TableGen def 记录 `S4_vxsubaddw`。
- **L990 EN**: Declares TableGen def record `S5_asrhub_rnd_sat_goodsyntax`.
  **L990 CN**: 声明 TableGen def 记录 `S5_asrhub_rnd_sat_goodsyntax`。
- **L991 EN**: Declares TableGen def record `S5_asrhub_sat`.
  **L991 CN**: 声明 TableGen def 记录 `S5_asrhub_sat`。
- **L992 EN**: Declares TableGen def record `S5_popcountp`.
  **L992 CN**: 声明 TableGen def 记录 `S5_popcountp`。
- **L993 EN**: Declares TableGen def record `S5_vasrhrnd_goodsyntax`.
  **L993 CN**: 声明 TableGen def 记录 `S5_vasrhrnd_goodsyntax`。
- **L994 EN**: Declares TableGen def record `Y2_dccleana`.
  **L994 CN**: 声明 TableGen def 记录 `Y2_dccleana`。
- **L995 EN**: Declares TableGen def record `Y2_dccleaninva`.
  **L995 CN**: 声明 TableGen def 记录 `Y2_dccleaninva`。
- **L996 EN**: Declares TableGen def record `Y2_dcfetch`.
  **L996 CN**: 声明 TableGen def 记录 `Y2_dcfetch`。
- **L997 EN**: Declares TableGen def record `Y2_dcinva`.
  **L997 CN**: 声明 TableGen def 记录 `Y2_dcinva`。
- **L998 EN**: Declares TableGen def record `Y2_dczeroa`.
  **L998 CN**: 声明 TableGen def 记录 `Y2_dczeroa`。
- **L999 EN**: Declares TableGen def record `Y4_l2fetch`.
  **L999 CN**: 声明 TableGen def 记录 `Y4_l2fetch`。
- **L1000 EN**: Declares TableGen def record `Y5_l2fetch`.
  **L1000 CN**: 声明 TableGen def 记录 `Y5_l2fetch`。
- **L1001 EN**: Blank line separating nearby declarations or logic blocks.
  **L1001 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1002 EN**: Comment explains nearby logic, constraints, or intent: `V60 Scalar Instructions.`.
  **L1002 CN**: 注释解释附近代码的逻辑、约束或设计意图：`V60 Scalar Instructions.`。
- **L1003 EN**: Blank line separating nearby declarations or logic blocks.
  **L1003 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1004 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = V60.Features in {`.
  **L1004 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = V60.Features in {`。
- **L1005 EN**: Declares TableGen def record `S6_rol_i_p`.
  **L1005 CN**: 声明 TableGen def 记录 `S6_rol_i_p`。
- **L1006 EN**: Declares TableGen def record `S6_rol_i_p_acc`.
  **L1006 CN**: 声明 TableGen def 记录 `S6_rol_i_p_acc`。
- **L1007 EN**: Declares TableGen def record `S6_rol_i_p_and`.
  **L1007 CN**: 声明 TableGen def 记录 `S6_rol_i_p_and`。
- **L1008 EN**: Declares TableGen def record `S6_rol_i_p_nac`.
  **L1008 CN**: 声明 TableGen def 记录 `S6_rol_i_p_nac`。

### Lines 1009-1036

````tablegen
  def S6_rol_i_p_or : HexagonBuiltin<"long long int(long long int, long long int, unsigned _Constant int)">;
  def S6_rol_i_p_xacc : HexagonBuiltin<"long long int(long long int, long long int, unsigned _Constant int)">;
  def S6_rol_i_r : HexagonBuiltin<"int(int, unsigned _Constant int)">;
  def S6_rol_i_r_acc : HexagonBuiltin<"int(int, int, unsigned _Constant int)">;
  def S6_rol_i_r_and : HexagonBuiltin<"int(int, int, unsigned _Constant int)">;
  def S6_rol_i_r_nac : HexagonBuiltin<"int(int, int, unsigned _Constant int)">;
  def S6_rol_i_r_or : HexagonBuiltin<"int(int, int, unsigned _Constant int)">;
  def S6_rol_i_r_xacc : HexagonBuiltin<"int(int, int, unsigned _Constant int)">;
}

// V62 Scalar Instructions.

let Features = V62.Features in {
  def M6_vabsdiffb : HexagonBuiltin<"long long int(long long int, long long int)">;
  def M6_vabsdiffub : HexagonBuiltin<"long long int(long long int, long long int)">;
  def S6_vsplatrbp : HexagonBuiltin<"long long int(int)">;
  def S6_vtrunehb_ppp : HexagonBuiltin<"long long int(long long int, long long int)">;
  def S6_vtrunohb_ppp : HexagonBuiltin<"long long int(long long int, long long int)">;
}

// V65 Scalar Instructions.

let Features = V65.Features in {
  def A6_vcmpbeq_notany : HexagonBuiltin<"int(long long int, long long int)">;
}

// V66 Scalar Instructions.

````
- **L1009 EN**: Declares TableGen def record `S6_rol_i_p_or`.
  **L1009 CN**: 声明 TableGen def 记录 `S6_rol_i_p_or`。
- **L1010 EN**: Declares TableGen def record `S6_rol_i_p_xacc`.
  **L1010 CN**: 声明 TableGen def 记录 `S6_rol_i_p_xacc`。
- **L1011 EN**: Declares TableGen def record `S6_rol_i_r`.
  **L1011 CN**: 声明 TableGen def 记录 `S6_rol_i_r`。
- **L1012 EN**: Declares TableGen def record `S6_rol_i_r_acc`.
  **L1012 CN**: 声明 TableGen def 记录 `S6_rol_i_r_acc`。
- **L1013 EN**: Declares TableGen def record `S6_rol_i_r_and`.
  **L1013 CN**: 声明 TableGen def 记录 `S6_rol_i_r_and`。
- **L1014 EN**: Declares TableGen def record `S6_rol_i_r_nac`.
  **L1014 CN**: 声明 TableGen def 记录 `S6_rol_i_r_nac`。
- **L1015 EN**: Declares TableGen def record `S6_rol_i_r_or`.
  **L1015 CN**: 声明 TableGen def 记录 `S6_rol_i_r_or`。
- **L1016 EN**: Declares TableGen def record `S6_rol_i_r_xacc`.
  **L1016 CN**: 声明 TableGen def 记录 `S6_rol_i_r_xacc`。
- **L1017 EN**: Closes the current lexical scope or compound statement.
  **L1017 CN**: 结束当前词法作用域或复合语句块。
- **L1018 EN**: Blank line separating nearby declarations or logic blocks.
  **L1018 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1019 EN**: Comment explains nearby logic, constraints, or intent: `V62 Scalar Instructions.`.
  **L1019 CN**: 注释解释附近代码的逻辑、约束或设计意图：`V62 Scalar Instructions.`。
- **L1020 EN**: Blank line separating nearby declarations or logic blocks.
  **L1020 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1021 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = V62.Features in {`.
  **L1021 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = V62.Features in {`。
- **L1022 EN**: Declares TableGen def record `M6_vabsdiffb`.
  **L1022 CN**: 声明 TableGen def 记录 `M6_vabsdiffb`。
- **L1023 EN**: Declares TableGen def record `M6_vabsdiffub`.
  **L1023 CN**: 声明 TableGen def 记录 `M6_vabsdiffub`。
- **L1024 EN**: Declares TableGen def record `S6_vsplatrbp`.
  **L1024 CN**: 声明 TableGen def 记录 `S6_vsplatrbp`。
- **L1025 EN**: Declares TableGen def record `S6_vtrunehb_ppp`.
  **L1025 CN**: 声明 TableGen def 记录 `S6_vtrunehb_ppp`。
- **L1026 EN**: Declares TableGen def record `S6_vtrunohb_ppp`.
  **L1026 CN**: 声明 TableGen def 记录 `S6_vtrunohb_ppp`。
- **L1027 EN**: Closes the current lexical scope or compound statement.
  **L1027 CN**: 结束当前词法作用域或复合语句块。
- **L1028 EN**: Blank line separating nearby declarations or logic blocks.
  **L1028 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1029 EN**: Comment explains nearby logic, constraints, or intent: `V65 Scalar Instructions.`.
  **L1029 CN**: 注释解释附近代码的逻辑、约束或设计意图：`V65 Scalar Instructions.`。
- **L1030 EN**: Blank line separating nearby declarations or logic blocks.
  **L1030 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1031 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = V65.Features in {`.
  **L1031 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = V65.Features in {`。
- **L1032 EN**: Declares TableGen def record `A6_vcmpbeq_notany`.
  **L1032 CN**: 声明 TableGen def 记录 `A6_vcmpbeq_notany`。
- **L1033 EN**: Closes the current lexical scope or compound statement.
  **L1033 CN**: 结束当前词法作用域或复合语句块。
- **L1034 EN**: Blank line separating nearby declarations or logic blocks.
  **L1034 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1035 EN**: Comment explains nearby logic, constraints, or intent: `V66 Scalar Instructions.`.
  **L1035 CN**: 注释解释附近代码的逻辑、约束或设计意图：`V66 Scalar Instructions.`。
- **L1036 EN**: Blank line separating nearby declarations or logic blocks.
  **L1036 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1037-1064

````tablegen
let Features = V66.Features in {
  def F2_dfadd : HexagonBuiltin<"double(double, double)">;
  def F2_dfsub : HexagonBuiltin<"double(double, double)">;
  def M2_mnaci : HexagonBuiltin<"int(int, int, int)">;
  def S2_mask : HexagonBuiltin<"int(unsigned _Constant int, unsigned _Constant int)">;
}

// V67 Scalar Instructions.

let Features = "audio" in {
  def A7_clip : HexagonBuiltin<"int(int, unsigned _Constant int)">;
  def A7_croundd_ri : HexagonBuiltin<"long long int(long long int, unsigned _Constant int)">;
  def A7_croundd_rr : HexagonBuiltin<"long long int(long long int, int)">;
  def A7_vclip : HexagonBuiltin<"long long int(long long int, unsigned _Constant int)">;
}
let Features = V67.Features in {
  def F2_dfmax : HexagonBuiltin<"double(double, double)">;
  def F2_dfmin : HexagonBuiltin<"double(double, double)">;
  def F2_dfmpyfix : HexagonBuiltin<"double(double, double)">;
  def F2_dfmpyhh : HexagonBuiltin<"double(double, double, double)">;
  def F2_dfmpylh : HexagonBuiltin<"double(double, double, double)">;
  def F2_dfmpyll : HexagonBuiltin<"double(double, double)">;
}
let Features = "audio" in {
  def M7_dcmpyiw : HexagonBuiltin<"long long int(long long int, long long int)">;
  def M7_dcmpyiw_acc : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
  def M7_dcmpyiwc : HexagonBuiltin<"long long int(long long int, long long int)">;
  def M7_dcmpyiwc_acc : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
````
- **L1037 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = V66.Features in {`.
  **L1037 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = V66.Features in {`。
- **L1038 EN**: Declares TableGen def record `F2_dfadd`.
  **L1038 CN**: 声明 TableGen def 记录 `F2_dfadd`。
- **L1039 EN**: Declares TableGen def record `F2_dfsub`.
  **L1039 CN**: 声明 TableGen def 记录 `F2_dfsub`。
- **L1040 EN**: Declares TableGen def record `M2_mnaci`.
  **L1040 CN**: 声明 TableGen def 记录 `M2_mnaci`。
- **L1041 EN**: Declares TableGen def record `S2_mask`.
  **L1041 CN**: 声明 TableGen def 记录 `S2_mask`。
- **L1042 EN**: Closes the current lexical scope or compound statement.
  **L1042 CN**: 结束当前词法作用域或复合语句块。
- **L1043 EN**: Blank line separating nearby declarations or logic blocks.
  **L1043 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1044 EN**: Comment explains nearby logic, constraints, or intent: `V67 Scalar Instructions.`.
  **L1044 CN**: 注释解释附近代码的逻辑、约束或设计意图：`V67 Scalar Instructions.`。
- **L1045 EN**: Blank line separating nearby declarations or logic blocks.
  **L1045 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1046 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "audio" in {`.
  **L1046 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "audio" in {`。
- **L1047 EN**: Declares TableGen def record `A7_clip`.
  **L1047 CN**: 声明 TableGen def 记录 `A7_clip`。
- **L1048 EN**: Declares TableGen def record `A7_croundd_ri`.
  **L1048 CN**: 声明 TableGen def 记录 `A7_croundd_ri`。
- **L1049 EN**: Declares TableGen def record `A7_croundd_rr`.
  **L1049 CN**: 声明 TableGen def 记录 `A7_croundd_rr`。
- **L1050 EN**: Declares TableGen def record `A7_vclip`.
  **L1050 CN**: 声明 TableGen def 记录 `A7_vclip`。
- **L1051 EN**: Closes the current lexical scope or compound statement.
  **L1051 CN**: 结束当前词法作用域或复合语句块。
- **L1052 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = V67.Features in {`.
  **L1052 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = V67.Features in {`。
- **L1053 EN**: Declares TableGen def record `F2_dfmax`.
  **L1053 CN**: 声明 TableGen def 记录 `F2_dfmax`。
- **L1054 EN**: Declares TableGen def record `F2_dfmin`.
  **L1054 CN**: 声明 TableGen def 记录 `F2_dfmin`。
- **L1055 EN**: Declares TableGen def record `F2_dfmpyfix`.
  **L1055 CN**: 声明 TableGen def 记录 `F2_dfmpyfix`。
- **L1056 EN**: Declares TableGen def record `F2_dfmpyhh`.
  **L1056 CN**: 声明 TableGen def 记录 `F2_dfmpyhh`。
- **L1057 EN**: Declares TableGen def record `F2_dfmpylh`.
  **L1057 CN**: 声明 TableGen def 记录 `F2_dfmpylh`。
- **L1058 EN**: Declares TableGen def record `F2_dfmpyll`.
  **L1058 CN**: 声明 TableGen def 记录 `F2_dfmpyll`。
- **L1059 EN**: Closes the current lexical scope or compound statement.
  **L1059 CN**: 结束当前词法作用域或复合语句块。
- **L1060 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "audio" in {`.
  **L1060 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "audio" in {`。
- **L1061 EN**: Declares TableGen def record `M7_dcmpyiw`.
  **L1061 CN**: 声明 TableGen def 记录 `M7_dcmpyiw`。
- **L1062 EN**: Declares TableGen def record `M7_dcmpyiw_acc`.
  **L1062 CN**: 声明 TableGen def 记录 `M7_dcmpyiw_acc`。
- **L1063 EN**: Declares TableGen def record `M7_dcmpyiwc`.
  **L1063 CN**: 声明 TableGen def 记录 `M7_dcmpyiwc`。
- **L1064 EN**: Declares TableGen def record `M7_dcmpyiwc_acc`.
  **L1064 CN**: 声明 TableGen def 记录 `M7_dcmpyiwc_acc`。

### Lines 1065-1092

````tablegen
  def M7_dcmpyrw : HexagonBuiltin<"long long int(long long int, long long int)">;
  def M7_dcmpyrw_acc : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
  def M7_dcmpyrwc : HexagonBuiltin<"long long int(long long int, long long int)">;
  def M7_dcmpyrwc_acc : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
}
let Features = V67.Features in {
  def M7_vdmpy : HexagonBuiltin<"long long int(long long int, long long int)">;
  def M7_vdmpy_acc : HexagonBuiltin<"long long int(long long int, long long int, long long int)">;
}
let Features = "audio" in {
  def M7_wcmpyiw : HexagonBuiltin<"int(long long int, long long int)">;
  def M7_wcmpyiw_rnd : HexagonBuiltin<"int(long long int, long long int)">;
  def M7_wcmpyiwc : HexagonBuiltin<"int(long long int, long long int)">;
  def M7_wcmpyiwc_rnd : HexagonBuiltin<"int(long long int, long long int)">;
  def M7_wcmpyrw : HexagonBuiltin<"int(long long int, long long int)">;
  def M7_wcmpyrw_rnd : HexagonBuiltin<"int(long long int, long long int)">;
  def M7_wcmpyrwc : HexagonBuiltin<"int(long long int, long long int)">;
  def M7_wcmpyrwc_rnd : HexagonBuiltin<"int(long long int, long long int)">;
}

// V68 Scalar Instructions.

let Features = V68.Features in {
  def Y6_dmlink : HexagonBuiltin<"void(void *, void *)">;
  def Y6_dmpause : HexagonBuiltin<"int()">;
  def Y6_dmpoll : HexagonBuiltin<"int()">;
  def Y6_dmresume : HexagonBuiltin<"void(void *)">;
  def Y6_dmstart : HexagonBuiltin<"void(void *)">;
````
- **L1065 EN**: Declares TableGen def record `M7_dcmpyrw`.
  **L1065 CN**: 声明 TableGen def 记录 `M7_dcmpyrw`。
- **L1066 EN**: Declares TableGen def record `M7_dcmpyrw_acc`.
  **L1066 CN**: 声明 TableGen def 记录 `M7_dcmpyrw_acc`。
- **L1067 EN**: Declares TableGen def record `M7_dcmpyrwc`.
  **L1067 CN**: 声明 TableGen def 记录 `M7_dcmpyrwc`。
- **L1068 EN**: Declares TableGen def record `M7_dcmpyrwc_acc`.
  **L1068 CN**: 声明 TableGen def 记录 `M7_dcmpyrwc_acc`。
- **L1069 EN**: Closes the current lexical scope or compound statement.
  **L1069 CN**: 结束当前词法作用域或复合语句块。
- **L1070 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = V67.Features in {`.
  **L1070 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = V67.Features in {`。
- **L1071 EN**: Declares TableGen def record `M7_vdmpy`.
  **L1071 CN**: 声明 TableGen def 记录 `M7_vdmpy`。
- **L1072 EN**: Declares TableGen def record `M7_vdmpy_acc`.
  **L1072 CN**: 声明 TableGen def 记录 `M7_vdmpy_acc`。
- **L1073 EN**: Closes the current lexical scope or compound statement.
  **L1073 CN**: 结束当前词法作用域或复合语句块。
- **L1074 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "audio" in {`.
  **L1074 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "audio" in {`。
- **L1075 EN**: Declares TableGen def record `M7_wcmpyiw`.
  **L1075 CN**: 声明 TableGen def 记录 `M7_wcmpyiw`。
- **L1076 EN**: Declares TableGen def record `M7_wcmpyiw_rnd`.
  **L1076 CN**: 声明 TableGen def 记录 `M7_wcmpyiw_rnd`。
- **L1077 EN**: Declares TableGen def record `M7_wcmpyiwc`.
  **L1077 CN**: 声明 TableGen def 记录 `M7_wcmpyiwc`。
- **L1078 EN**: Declares TableGen def record `M7_wcmpyiwc_rnd`.
  **L1078 CN**: 声明 TableGen def 记录 `M7_wcmpyiwc_rnd`。
- **L1079 EN**: Declares TableGen def record `M7_wcmpyrw`.
  **L1079 CN**: 声明 TableGen def 记录 `M7_wcmpyrw`。
- **L1080 EN**: Declares TableGen def record `M7_wcmpyrw_rnd`.
  **L1080 CN**: 声明 TableGen def 记录 `M7_wcmpyrw_rnd`。
- **L1081 EN**: Declares TableGen def record `M7_wcmpyrwc`.
  **L1081 CN**: 声明 TableGen def 记录 `M7_wcmpyrwc`。
- **L1082 EN**: Declares TableGen def record `M7_wcmpyrwc_rnd`.
  **L1082 CN**: 声明 TableGen def 记录 `M7_wcmpyrwc_rnd`。
- **L1083 EN**: Closes the current lexical scope or compound statement.
  **L1083 CN**: 结束当前词法作用域或复合语句块。
- **L1084 EN**: Blank line separating nearby declarations or logic blocks.
  **L1084 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1085 EN**: Comment explains nearby logic, constraints, or intent: `V68 Scalar Instructions.`.
  **L1085 CN**: 注释解释附近代码的逻辑、约束或设计意图：`V68 Scalar Instructions.`。
- **L1086 EN**: Blank line separating nearby declarations or logic blocks.
  **L1086 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1087 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = V68.Features in {`.
  **L1087 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = V68.Features in {`。
- **L1088 EN**: Declares TableGen def record `Y6_dmlink`.
  **L1088 CN**: 声明 TableGen def 记录 `Y6_dmlink`。
- **L1089 EN**: Declares TableGen def record `Y6_dmpause`.
  **L1089 CN**: 声明 TableGen def 记录 `Y6_dmpause`。
- **L1090 EN**: Declares TableGen def record `Y6_dmpoll`.
  **L1090 CN**: 声明 TableGen def 记录 `Y6_dmpoll`。
- **L1091 EN**: Declares TableGen def record `Y6_dmresume`.
  **L1091 CN**: 声明 TableGen def 记录 `Y6_dmresume`。
- **L1092 EN**: Declares TableGen def record `Y6_dmstart`.
  **L1092 CN**: 声明 TableGen def 记录 `Y6_dmstart`。

### Lines 1093-1120

````tablegen
  def Y6_dmwait : HexagonBuiltin<"int()">;
}

// V60 HVX Instructions.

let Features = HVXV60.Features in {
  def V6_extractw : HexagonBuiltin<"int(_Vector<16, int>, int)">;
  def V6_extractw_128B : HexagonBuiltin<"int(_Vector<32, int>, int)">;
  def V6_hi : HexagonBuiltin<"_Vector<16, int>(_Vector<32, int>)">;
  def V6_hi_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<64, int>)">;
  def V6_lo : HexagonBuiltin<"_Vector<16, int>(_Vector<32, int>)">;
  def V6_lo_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<64, int>)">;
  def V6_lvsplatw : HexagonBuiltin<"_Vector<16, int>(int)">;
  def V6_lvsplatw_128B : HexagonBuiltin<"_Vector<32, int>(int)">;
  def V6_pred_and : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<64, bool>)">;
  def V6_pred_and_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<128, bool>)">;
  def V6_pred_and_n : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<64, bool>)">;
  def V6_pred_and_n_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<128, bool>)">;
  def V6_pred_not : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>)">;
  def V6_pred_not_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>)">;
  def V6_pred_or : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<64, bool>)">;
  def V6_pred_or_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<128, bool>)">;
  def V6_pred_or_n : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<64, bool>)">;
  def V6_pred_or_n_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<128, bool>)">;
  def V6_pred_scalar2 : HexagonBuiltin<"_Vector<64, bool>(int)">;
  def V6_pred_scalar2_128B : HexagonBuiltin<"_Vector<128, bool>(int)">;
  def V6_pred_xor : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<64, bool>)">;
  def V6_pred_xor_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<128, bool>)">;
````
- **L1093 EN**: Declares TableGen def record `Y6_dmwait`.
  **L1093 CN**: 声明 TableGen def 记录 `Y6_dmwait`。
- **L1094 EN**: Closes the current lexical scope or compound statement.
  **L1094 CN**: 结束当前词法作用域或复合语句块。
- **L1095 EN**: Blank line separating nearby declarations or logic blocks.
  **L1095 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1096 EN**: Comment explains nearby logic, constraints, or intent: `V60 HVX Instructions.`.
  **L1096 CN**: 注释解释附近代码的逻辑、约束或设计意图：`V60 HVX Instructions.`。
- **L1097 EN**: Blank line separating nearby declarations or logic blocks.
  **L1097 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1098 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = HVXV60.Features in {`.
  **L1098 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = HVXV60.Features in {`。
- **L1099 EN**: Declares TableGen def record `V6_extractw`.
  **L1099 CN**: 声明 TableGen def 记录 `V6_extractw`。
- **L1100 EN**: Declares TableGen def record `V6_extractw_128B`.
  **L1100 CN**: 声明 TableGen def 记录 `V6_extractw_128B`。
- **L1101 EN**: Declares TableGen def record `V6_hi`.
  **L1101 CN**: 声明 TableGen def 记录 `V6_hi`。
- **L1102 EN**: Declares TableGen def record `V6_hi_128B`.
  **L1102 CN**: 声明 TableGen def 记录 `V6_hi_128B`。
- **L1103 EN**: Declares TableGen def record `V6_lo`.
  **L1103 CN**: 声明 TableGen def 记录 `V6_lo`。
- **L1104 EN**: Declares TableGen def record `V6_lo_128B`.
  **L1104 CN**: 声明 TableGen def 记录 `V6_lo_128B`。
- **L1105 EN**: Declares TableGen def record `V6_lvsplatw`.
  **L1105 CN**: 声明 TableGen def 记录 `V6_lvsplatw`。
- **L1106 EN**: Declares TableGen def record `V6_lvsplatw_128B`.
  **L1106 CN**: 声明 TableGen def 记录 `V6_lvsplatw_128B`。
- **L1107 EN**: Declares TableGen def record `V6_pred_and`.
  **L1107 CN**: 声明 TableGen def 记录 `V6_pred_and`。
- **L1108 EN**: Declares TableGen def record `V6_pred_and_128B`.
  **L1108 CN**: 声明 TableGen def 记录 `V6_pred_and_128B`。
- **L1109 EN**: Declares TableGen def record `V6_pred_and_n`.
  **L1109 CN**: 声明 TableGen def 记录 `V6_pred_and_n`。
- **L1110 EN**: Declares TableGen def record `V6_pred_and_n_128B`.
  **L1110 CN**: 声明 TableGen def 记录 `V6_pred_and_n_128B`。
- **L1111 EN**: Declares TableGen def record `V6_pred_not`.
  **L1111 CN**: 声明 TableGen def 记录 `V6_pred_not`。
- **L1112 EN**: Declares TableGen def record `V6_pred_not_128B`.
  **L1112 CN**: 声明 TableGen def 记录 `V6_pred_not_128B`。
- **L1113 EN**: Declares TableGen def record `V6_pred_or`.
  **L1113 CN**: 声明 TableGen def 记录 `V6_pred_or`。
- **L1114 EN**: Declares TableGen def record `V6_pred_or_128B`.
  **L1114 CN**: 声明 TableGen def 记录 `V6_pred_or_128B`。
- **L1115 EN**: Declares TableGen def record `V6_pred_or_n`.
  **L1115 CN**: 声明 TableGen def 记录 `V6_pred_or_n`。
- **L1116 EN**: Declares TableGen def record `V6_pred_or_n_128B`.
  **L1116 CN**: 声明 TableGen def 记录 `V6_pred_or_n_128B`。
- **L1117 EN**: Declares TableGen def record `V6_pred_scalar2`.
  **L1117 CN**: 声明 TableGen def 记录 `V6_pred_scalar2`。
- **L1118 EN**: Declares TableGen def record `V6_pred_scalar2_128B`.
  **L1118 CN**: 声明 TableGen def 记录 `V6_pred_scalar2_128B`。
- **L1119 EN**: Declares TableGen def record `V6_pred_xor`.
  **L1119 CN**: 声明 TableGen def 记录 `V6_pred_xor`。
- **L1120 EN**: Declares TableGen def record `V6_pred_xor_128B`.
  **L1120 CN**: 声明 TableGen def 记录 `V6_pred_xor_128B`。

### Lines 1121-1148

````tablegen
  def V6_vS32b_nqpred_ai : HexagonBuiltin<"void(_Vector<64, bool>, void *, _Vector<16, int>)">;
  def V6_vS32b_nqpred_ai_128B : HexagonBuiltin<"void(_Vector<128, bool>, void *, _Vector<32, int>)">;
  def V6_vS32b_nt_nqpred_ai : HexagonBuiltin<"void(_Vector<64, bool>, void *, _Vector<16, int>)">;
  def V6_vS32b_nt_nqpred_ai_128B : HexagonBuiltin<"void(_Vector<128, bool>, void *, _Vector<32, int>)">;
  def V6_vS32b_nt_qpred_ai : HexagonBuiltin<"void(_Vector<64, bool>, void *, _Vector<16, int>)">;
  def V6_vS32b_nt_qpred_ai_128B : HexagonBuiltin<"void(_Vector<128, bool>, void *, _Vector<32, int>)">;
  def V6_vS32b_qpred_ai : HexagonBuiltin<"void(_Vector<64, bool>, void *, _Vector<16, int>)">;
  def V6_vS32b_qpred_ai_128B : HexagonBuiltin<"void(_Vector<128, bool>, void *, _Vector<32, int>)">;
  def V6_vabsdiffh : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vabsdiffh_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vabsdiffub : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vabsdiffub_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vabsdiffuh : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vabsdiffuh_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vabsdiffw : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vabsdiffw_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vabsh : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vabsh_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vabsh_sat : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vabsh_sat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vabsw : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vabsw_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vabsw_sat : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vabsw_sat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vaddb : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vaddb_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vaddb_dv : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vaddb_dv_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<64, int>)">;
````
- **L1121 EN**: Declares TableGen def record `V6_vS32b_nqpred_ai`.
  **L1121 CN**: 声明 TableGen def 记录 `V6_vS32b_nqpred_ai`。
- **L1122 EN**: Declares TableGen def record `V6_vS32b_nqpred_ai_128B`.
  **L1122 CN**: 声明 TableGen def 记录 `V6_vS32b_nqpred_ai_128B`。
- **L1123 EN**: Declares TableGen def record `V6_vS32b_nt_nqpred_ai`.
  **L1123 CN**: 声明 TableGen def 记录 `V6_vS32b_nt_nqpred_ai`。
- **L1124 EN**: Declares TableGen def record `V6_vS32b_nt_nqpred_ai_128B`.
  **L1124 CN**: 声明 TableGen def 记录 `V6_vS32b_nt_nqpred_ai_128B`。
- **L1125 EN**: Declares TableGen def record `V6_vS32b_nt_qpred_ai`.
  **L1125 CN**: 声明 TableGen def 记录 `V6_vS32b_nt_qpred_ai`。
- **L1126 EN**: Declares TableGen def record `V6_vS32b_nt_qpred_ai_128B`.
  **L1126 CN**: 声明 TableGen def 记录 `V6_vS32b_nt_qpred_ai_128B`。
- **L1127 EN**: Declares TableGen def record `V6_vS32b_qpred_ai`.
  **L1127 CN**: 声明 TableGen def 记录 `V6_vS32b_qpred_ai`。
- **L1128 EN**: Declares TableGen def record `V6_vS32b_qpred_ai_128B`.
  **L1128 CN**: 声明 TableGen def 记录 `V6_vS32b_qpred_ai_128B`。
- **L1129 EN**: Declares TableGen def record `V6_vabsdiffh`.
  **L1129 CN**: 声明 TableGen def 记录 `V6_vabsdiffh`。
- **L1130 EN**: Declares TableGen def record `V6_vabsdiffh_128B`.
  **L1130 CN**: 声明 TableGen def 记录 `V6_vabsdiffh_128B`。
- **L1131 EN**: Declares TableGen def record `V6_vabsdiffub`.
  **L1131 CN**: 声明 TableGen def 记录 `V6_vabsdiffub`。
- **L1132 EN**: Declares TableGen def record `V6_vabsdiffub_128B`.
  **L1132 CN**: 声明 TableGen def 记录 `V6_vabsdiffub_128B`。
- **L1133 EN**: Declares TableGen def record `V6_vabsdiffuh`.
  **L1133 CN**: 声明 TableGen def 记录 `V6_vabsdiffuh`。
- **L1134 EN**: Declares TableGen def record `V6_vabsdiffuh_128B`.
  **L1134 CN**: 声明 TableGen def 记录 `V6_vabsdiffuh_128B`。
- **L1135 EN**: Declares TableGen def record `V6_vabsdiffw`.
  **L1135 CN**: 声明 TableGen def 记录 `V6_vabsdiffw`。
- **L1136 EN**: Declares TableGen def record `V6_vabsdiffw_128B`.
  **L1136 CN**: 声明 TableGen def 记录 `V6_vabsdiffw_128B`。
- **L1137 EN**: Declares TableGen def record `V6_vabsh`.
  **L1137 CN**: 声明 TableGen def 记录 `V6_vabsh`。
- **L1138 EN**: Declares TableGen def record `V6_vabsh_128B`.
  **L1138 CN**: 声明 TableGen def 记录 `V6_vabsh_128B`。
- **L1139 EN**: Declares TableGen def record `V6_vabsh_sat`.
  **L1139 CN**: 声明 TableGen def 记录 `V6_vabsh_sat`。
- **L1140 EN**: Declares TableGen def record `V6_vabsh_sat_128B`.
  **L1140 CN**: 声明 TableGen def 记录 `V6_vabsh_sat_128B`。
- **L1141 EN**: Declares TableGen def record `V6_vabsw`.
  **L1141 CN**: 声明 TableGen def 记录 `V6_vabsw`。
- **L1142 EN**: Declares TableGen def record `V6_vabsw_128B`.
  **L1142 CN**: 声明 TableGen def 记录 `V6_vabsw_128B`。
- **L1143 EN**: Declares TableGen def record `V6_vabsw_sat`.
  **L1143 CN**: 声明 TableGen def 记录 `V6_vabsw_sat`。
- **L1144 EN**: Declares TableGen def record `V6_vabsw_sat_128B`.
  **L1144 CN**: 声明 TableGen def 记录 `V6_vabsw_sat_128B`。
- **L1145 EN**: Declares TableGen def record `V6_vaddb`.
  **L1145 CN**: 声明 TableGen def 记录 `V6_vaddb`。
- **L1146 EN**: Declares TableGen def record `V6_vaddb_128B`.
  **L1146 CN**: 声明 TableGen def 记录 `V6_vaddb_128B`。
- **L1147 EN**: Declares TableGen def record `V6_vaddb_dv`.
  **L1147 CN**: 声明 TableGen def 记录 `V6_vaddb_dv`。
- **L1148 EN**: Declares TableGen def record `V6_vaddb_dv_128B`.
  **L1148 CN**: 声明 TableGen def 记录 `V6_vaddb_dv_128B`。

### Lines 1149-1176

````tablegen
  def V6_vaddbnq : HexagonBuiltin<"_Vector<16, int>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vaddbnq_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vaddbq : HexagonBuiltin<"_Vector<16, int>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vaddbq_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vaddh : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vaddh_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vaddh_dv : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vaddh_dv_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<64, int>)">;
  def V6_vaddhnq : HexagonBuiltin<"_Vector<16, int>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vaddhnq_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vaddhq : HexagonBuiltin<"_Vector<16, int>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vaddhq_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vaddhsat : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vaddhsat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vaddhsat_dv : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vaddhsat_dv_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<64, int>)">;
  def V6_vaddhw : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vaddhw_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vaddubh : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vaddubh_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vaddubsat : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vaddubsat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vaddubsat_dv : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vaddubsat_dv_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<64, int>)">;
  def V6_vadduhsat : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vadduhsat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vadduhsat_dv : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vadduhsat_dv_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<64, int>)">;
````
- **L1149 EN**: Declares TableGen def record `V6_vaddbnq`.
  **L1149 CN**: 声明 TableGen def 记录 `V6_vaddbnq`。
- **L1150 EN**: Declares TableGen def record `V6_vaddbnq_128B`.
  **L1150 CN**: 声明 TableGen def 记录 `V6_vaddbnq_128B`。
- **L1151 EN**: Declares TableGen def record `V6_vaddbq`.
  **L1151 CN**: 声明 TableGen def 记录 `V6_vaddbq`。
- **L1152 EN**: Declares TableGen def record `V6_vaddbq_128B`.
  **L1152 CN**: 声明 TableGen def 记录 `V6_vaddbq_128B`。
- **L1153 EN**: Declares TableGen def record `V6_vaddh`.
  **L1153 CN**: 声明 TableGen def 记录 `V6_vaddh`。
- **L1154 EN**: Declares TableGen def record `V6_vaddh_128B`.
  **L1154 CN**: 声明 TableGen def 记录 `V6_vaddh_128B`。
- **L1155 EN**: Declares TableGen def record `V6_vaddh_dv`.
  **L1155 CN**: 声明 TableGen def 记录 `V6_vaddh_dv`。
- **L1156 EN**: Declares TableGen def record `V6_vaddh_dv_128B`.
  **L1156 CN**: 声明 TableGen def 记录 `V6_vaddh_dv_128B`。
- **L1157 EN**: Declares TableGen def record `V6_vaddhnq`.
  **L1157 CN**: 声明 TableGen def 记录 `V6_vaddhnq`。
- **L1158 EN**: Declares TableGen def record `V6_vaddhnq_128B`.
  **L1158 CN**: 声明 TableGen def 记录 `V6_vaddhnq_128B`。
- **L1159 EN**: Declares TableGen def record `V6_vaddhq`.
  **L1159 CN**: 声明 TableGen def 记录 `V6_vaddhq`。
- **L1160 EN**: Declares TableGen def record `V6_vaddhq_128B`.
  **L1160 CN**: 声明 TableGen def 记录 `V6_vaddhq_128B`。
- **L1161 EN**: Declares TableGen def record `V6_vaddhsat`.
  **L1161 CN**: 声明 TableGen def 记录 `V6_vaddhsat`。
- **L1162 EN**: Declares TableGen def record `V6_vaddhsat_128B`.
  **L1162 CN**: 声明 TableGen def 记录 `V6_vaddhsat_128B`。
- **L1163 EN**: Declares TableGen def record `V6_vaddhsat_dv`.
  **L1163 CN**: 声明 TableGen def 记录 `V6_vaddhsat_dv`。
- **L1164 EN**: Declares TableGen def record `V6_vaddhsat_dv_128B`.
  **L1164 CN**: 声明 TableGen def 记录 `V6_vaddhsat_dv_128B`。
- **L1165 EN**: Declares TableGen def record `V6_vaddhw`.
  **L1165 CN**: 声明 TableGen def 记录 `V6_vaddhw`。
- **L1166 EN**: Declares TableGen def record `V6_vaddhw_128B`.
  **L1166 CN**: 声明 TableGen def 记录 `V6_vaddhw_128B`。
- **L1167 EN**: Declares TableGen def record `V6_vaddubh`.
  **L1167 CN**: 声明 TableGen def 记录 `V6_vaddubh`。
- **L1168 EN**: Declares TableGen def record `V6_vaddubh_128B`.
  **L1168 CN**: 声明 TableGen def 记录 `V6_vaddubh_128B`。
- **L1169 EN**: Declares TableGen def record `V6_vaddubsat`.
  **L1169 CN**: 声明 TableGen def 记录 `V6_vaddubsat`。
- **L1170 EN**: Declares TableGen def record `V6_vaddubsat_128B`.
  **L1170 CN**: 声明 TableGen def 记录 `V6_vaddubsat_128B`。
- **L1171 EN**: Declares TableGen def record `V6_vaddubsat_dv`.
  **L1171 CN**: 声明 TableGen def 记录 `V6_vaddubsat_dv`。
- **L1172 EN**: Declares TableGen def record `V6_vaddubsat_dv_128B`.
  **L1172 CN**: 声明 TableGen def 记录 `V6_vaddubsat_dv_128B`。
- **L1173 EN**: Declares TableGen def record `V6_vadduhsat`.
  **L1173 CN**: 声明 TableGen def 记录 `V6_vadduhsat`。
- **L1174 EN**: Declares TableGen def record `V6_vadduhsat_128B`.
  **L1174 CN**: 声明 TableGen def 记录 `V6_vadduhsat_128B`。
- **L1175 EN**: Declares TableGen def record `V6_vadduhsat_dv`.
  **L1175 CN**: 声明 TableGen def 记录 `V6_vadduhsat_dv`。
- **L1176 EN**: Declares TableGen def record `V6_vadduhsat_dv_128B`.
  **L1176 CN**: 声明 TableGen def 记录 `V6_vadduhsat_dv_128B`。

### Lines 1177-1204

````tablegen
  def V6_vadduhw : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vadduhw_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vaddw : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vaddw_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vaddw_dv : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vaddw_dv_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<64, int>)">;
  def V6_vaddwnq : HexagonBuiltin<"_Vector<16, int>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vaddwnq_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vaddwq : HexagonBuiltin<"_Vector<16, int>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vaddwq_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vaddwsat : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vaddwsat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vaddwsat_dv : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vaddwsat_dv_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<64, int>)">;
  def V6_valignb : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, int)">;
  def V6_valignb_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_valignbi : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, unsigned _Constant int)">;
  def V6_valignbi_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, unsigned _Constant int)">;
  def V6_vand : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vand_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vandqrt : HexagonBuiltin<"_Vector<16, int>(_Vector<64, bool>, int)">;
  def V6_vandqrt_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<128, bool>, int)">;
  def V6_vandqrt_acc : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<64, bool>, int)">;
  def V6_vandqrt_acc_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<128, bool>, int)">;
  def V6_vandvrt : HexagonBuiltin<"_Vector<64, bool>(_Vector<16, int>, int)">;
  def V6_vandvrt_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<32, int>, int)">;
  def V6_vandvrt_acc : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<16, int>, int)">;
  def V6_vandvrt_acc_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<32, int>, int)">;
````
- **L1177 EN**: Declares TableGen def record `V6_vadduhw`.
  **L1177 CN**: 声明 TableGen def 记录 `V6_vadduhw`。
- **L1178 EN**: Declares TableGen def record `V6_vadduhw_128B`.
  **L1178 CN**: 声明 TableGen def 记录 `V6_vadduhw_128B`。
- **L1179 EN**: Declares TableGen def record `V6_vaddw`.
  **L1179 CN**: 声明 TableGen def 记录 `V6_vaddw`。
- **L1180 EN**: Declares TableGen def record `V6_vaddw_128B`.
  **L1180 CN**: 声明 TableGen def 记录 `V6_vaddw_128B`。
- **L1181 EN**: Declares TableGen def record `V6_vaddw_dv`.
  **L1181 CN**: 声明 TableGen def 记录 `V6_vaddw_dv`。
- **L1182 EN**: Declares TableGen def record `V6_vaddw_dv_128B`.
  **L1182 CN**: 声明 TableGen def 记录 `V6_vaddw_dv_128B`。
- **L1183 EN**: Declares TableGen def record `V6_vaddwnq`.
  **L1183 CN**: 声明 TableGen def 记录 `V6_vaddwnq`。
- **L1184 EN**: Declares TableGen def record `V6_vaddwnq_128B`.
  **L1184 CN**: 声明 TableGen def 记录 `V6_vaddwnq_128B`。
- **L1185 EN**: Declares TableGen def record `V6_vaddwq`.
  **L1185 CN**: 声明 TableGen def 记录 `V6_vaddwq`。
- **L1186 EN**: Declares TableGen def record `V6_vaddwq_128B`.
  **L1186 CN**: 声明 TableGen def 记录 `V6_vaddwq_128B`。
- **L1187 EN**: Declares TableGen def record `V6_vaddwsat`.
  **L1187 CN**: 声明 TableGen def 记录 `V6_vaddwsat`。
- **L1188 EN**: Declares TableGen def record `V6_vaddwsat_128B`.
  **L1188 CN**: 声明 TableGen def 记录 `V6_vaddwsat_128B`。
- **L1189 EN**: Declares TableGen def record `V6_vaddwsat_dv`.
  **L1189 CN**: 声明 TableGen def 记录 `V6_vaddwsat_dv`。
- **L1190 EN**: Declares TableGen def record `V6_vaddwsat_dv_128B`.
  **L1190 CN**: 声明 TableGen def 记录 `V6_vaddwsat_dv_128B`。
- **L1191 EN**: Declares TableGen def record `V6_valignb`.
  **L1191 CN**: 声明 TableGen def 记录 `V6_valignb`。
- **L1192 EN**: Declares TableGen def record `V6_valignb_128B`.
  **L1192 CN**: 声明 TableGen def 记录 `V6_valignb_128B`。
- **L1193 EN**: Declares TableGen def record `V6_valignbi`.
  **L1193 CN**: 声明 TableGen def 记录 `V6_valignbi`。
- **L1194 EN**: Declares TableGen def record `V6_valignbi_128B`.
  **L1194 CN**: 声明 TableGen def 记录 `V6_valignbi_128B`。
- **L1195 EN**: Declares TableGen def record `V6_vand`.
  **L1195 CN**: 声明 TableGen def 记录 `V6_vand`。
- **L1196 EN**: Declares TableGen def record `V6_vand_128B`.
  **L1196 CN**: 声明 TableGen def 记录 `V6_vand_128B`。
- **L1197 EN**: Declares TableGen def record `V6_vandqrt`.
  **L1197 CN**: 声明 TableGen def 记录 `V6_vandqrt`。
- **L1198 EN**: Declares TableGen def record `V6_vandqrt_128B`.
  **L1198 CN**: 声明 TableGen def 记录 `V6_vandqrt_128B`。
- **L1199 EN**: Declares TableGen def record `V6_vandqrt_acc`.
  **L1199 CN**: 声明 TableGen def 记录 `V6_vandqrt_acc`。
- **L1200 EN**: Declares TableGen def record `V6_vandqrt_acc_128B`.
  **L1200 CN**: 声明 TableGen def 记录 `V6_vandqrt_acc_128B`。
- **L1201 EN**: Declares TableGen def record `V6_vandvrt`.
  **L1201 CN**: 声明 TableGen def 记录 `V6_vandvrt`。
- **L1202 EN**: Declares TableGen def record `V6_vandvrt_128B`.
  **L1202 CN**: 声明 TableGen def 记录 `V6_vandvrt_128B`。
- **L1203 EN**: Declares TableGen def record `V6_vandvrt_acc`.
  **L1203 CN**: 声明 TableGen def 记录 `V6_vandvrt_acc`。
- **L1204 EN**: Declares TableGen def record `V6_vandvrt_acc_128B`.
  **L1204 CN**: 声明 TableGen def 记录 `V6_vandvrt_acc_128B`。

### Lines 1205-1232

````tablegen
  def V6_vaslh : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, int)">;
  def V6_vaslh_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, int)">;
  def V6_vaslhv : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vaslhv_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vaslw : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, int)">;
  def V6_vaslw_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, int)">;
  def V6_vaslw_acc : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, int)">;
  def V6_vaslw_acc_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vaslwv : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vaslwv_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vasrh : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, int)">;
  def V6_vasrh_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, int)">;
  def V6_vasrhbrndsat : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, int)">;
  def V6_vasrhbrndsat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vasrhubrndsat : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, int)">;
  def V6_vasrhubrndsat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vasrhubsat : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, int)">;
  def V6_vasrhubsat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vasrhv : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vasrhv_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vasrw : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, int)">;
  def V6_vasrw_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, int)">;
  def V6_vasrw_acc : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, int)">;
  def V6_vasrw_acc_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vasrwh : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, int)">;
  def V6_vasrwh_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vasrwhrndsat : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, int)">;
  def V6_vasrwhrndsat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
````
- **L1205 EN**: Declares TableGen def record `V6_vaslh`.
  **L1205 CN**: 声明 TableGen def 记录 `V6_vaslh`。
- **L1206 EN**: Declares TableGen def record `V6_vaslh_128B`.
  **L1206 CN**: 声明 TableGen def 记录 `V6_vaslh_128B`。
- **L1207 EN**: Declares TableGen def record `V6_vaslhv`.
  **L1207 CN**: 声明 TableGen def 记录 `V6_vaslhv`。
- **L1208 EN**: Declares TableGen def record `V6_vaslhv_128B`.
  **L1208 CN**: 声明 TableGen def 记录 `V6_vaslhv_128B`。
- **L1209 EN**: Declares TableGen def record `V6_vaslw`.
  **L1209 CN**: 声明 TableGen def 记录 `V6_vaslw`。
- **L1210 EN**: Declares TableGen def record `V6_vaslw_128B`.
  **L1210 CN**: 声明 TableGen def 记录 `V6_vaslw_128B`。
- **L1211 EN**: Declares TableGen def record `V6_vaslw_acc`.
  **L1211 CN**: 声明 TableGen def 记录 `V6_vaslw_acc`。
- **L1212 EN**: Declares TableGen def record `V6_vaslw_acc_128B`.
  **L1212 CN**: 声明 TableGen def 记录 `V6_vaslw_acc_128B`。
- **L1213 EN**: Declares TableGen def record `V6_vaslwv`.
  **L1213 CN**: 声明 TableGen def 记录 `V6_vaslwv`。
- **L1214 EN**: Declares TableGen def record `V6_vaslwv_128B`.
  **L1214 CN**: 声明 TableGen def 记录 `V6_vaslwv_128B`。
- **L1215 EN**: Declares TableGen def record `V6_vasrh`.
  **L1215 CN**: 声明 TableGen def 记录 `V6_vasrh`。
- **L1216 EN**: Declares TableGen def record `V6_vasrh_128B`.
  **L1216 CN**: 声明 TableGen def 记录 `V6_vasrh_128B`。
- **L1217 EN**: Declares TableGen def record `V6_vasrhbrndsat`.
  **L1217 CN**: 声明 TableGen def 记录 `V6_vasrhbrndsat`。
- **L1218 EN**: Declares TableGen def record `V6_vasrhbrndsat_128B`.
  **L1218 CN**: 声明 TableGen def 记录 `V6_vasrhbrndsat_128B`。
- **L1219 EN**: Declares TableGen def record `V6_vasrhubrndsat`.
  **L1219 CN**: 声明 TableGen def 记录 `V6_vasrhubrndsat`。
- **L1220 EN**: Declares TableGen def record `V6_vasrhubrndsat_128B`.
  **L1220 CN**: 声明 TableGen def 记录 `V6_vasrhubrndsat_128B`。
- **L1221 EN**: Declares TableGen def record `V6_vasrhubsat`.
  **L1221 CN**: 声明 TableGen def 记录 `V6_vasrhubsat`。
- **L1222 EN**: Declares TableGen def record `V6_vasrhubsat_128B`.
  **L1222 CN**: 声明 TableGen def 记录 `V6_vasrhubsat_128B`。
- **L1223 EN**: Declares TableGen def record `V6_vasrhv`.
  **L1223 CN**: 声明 TableGen def 记录 `V6_vasrhv`。
- **L1224 EN**: Declares TableGen def record `V6_vasrhv_128B`.
  **L1224 CN**: 声明 TableGen def 记录 `V6_vasrhv_128B`。
- **L1225 EN**: Declares TableGen def record `V6_vasrw`.
  **L1225 CN**: 声明 TableGen def 记录 `V6_vasrw`。
- **L1226 EN**: Declares TableGen def record `V6_vasrw_128B`.
  **L1226 CN**: 声明 TableGen def 记录 `V6_vasrw_128B`。
- **L1227 EN**: Declares TableGen def record `V6_vasrw_acc`.
  **L1227 CN**: 声明 TableGen def 记录 `V6_vasrw_acc`。
- **L1228 EN**: Declares TableGen def record `V6_vasrw_acc_128B`.
  **L1228 CN**: 声明 TableGen def 记录 `V6_vasrw_acc_128B`。
- **L1229 EN**: Declares TableGen def record `V6_vasrwh`.
  **L1229 CN**: 声明 TableGen def 记录 `V6_vasrwh`。
- **L1230 EN**: Declares TableGen def record `V6_vasrwh_128B`.
  **L1230 CN**: 声明 TableGen def 记录 `V6_vasrwh_128B`。
- **L1231 EN**: Declares TableGen def record `V6_vasrwhrndsat`.
  **L1231 CN**: 声明 TableGen def 记录 `V6_vasrwhrndsat`。
- **L1232 EN**: Declares TableGen def record `V6_vasrwhrndsat_128B`.
  **L1232 CN**: 声明 TableGen def 记录 `V6_vasrwhrndsat_128B`。

### Lines 1233-1260

````tablegen
  def V6_vasrwhsat : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, int)">;
  def V6_vasrwhsat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vasrwuhsat : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, int)">;
  def V6_vasrwuhsat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vasrwv : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vasrwv_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vassign : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vassign_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vassignp : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vassignp_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>)">;
  def V6_vavgh : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vavgh_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vavghrnd : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vavghrnd_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vavgub : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vavgub_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vavgubrnd : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vavgubrnd_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vavguh : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vavguh_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vavguhrnd : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vavguhrnd_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vavgw : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vavgw_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vavgwrnd : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vavgwrnd_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vcl0h : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vcl0h_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
````
- **L1233 EN**: Declares TableGen def record `V6_vasrwhsat`.
  **L1233 CN**: 声明 TableGen def 记录 `V6_vasrwhsat`。
- **L1234 EN**: Declares TableGen def record `V6_vasrwhsat_128B`.
  **L1234 CN**: 声明 TableGen def 记录 `V6_vasrwhsat_128B`。
- **L1235 EN**: Declares TableGen def record `V6_vasrwuhsat`.
  **L1235 CN**: 声明 TableGen def 记录 `V6_vasrwuhsat`。
- **L1236 EN**: Declares TableGen def record `V6_vasrwuhsat_128B`.
  **L1236 CN**: 声明 TableGen def 记录 `V6_vasrwuhsat_128B`。
- **L1237 EN**: Declares TableGen def record `V6_vasrwv`.
  **L1237 CN**: 声明 TableGen def 记录 `V6_vasrwv`。
- **L1238 EN**: Declares TableGen def record `V6_vasrwv_128B`.
  **L1238 CN**: 声明 TableGen def 记录 `V6_vasrwv_128B`。
- **L1239 EN**: Declares TableGen def record `V6_vassign`.
  **L1239 CN**: 声明 TableGen def 记录 `V6_vassign`。
- **L1240 EN**: Declares TableGen def record `V6_vassign_128B`.
  **L1240 CN**: 声明 TableGen def 记录 `V6_vassign_128B`。
- **L1241 EN**: Declares TableGen def record `V6_vassignp`.
  **L1241 CN**: 声明 TableGen def 记录 `V6_vassignp`。
- **L1242 EN**: Declares TableGen def record `V6_vassignp_128B`.
  **L1242 CN**: 声明 TableGen def 记录 `V6_vassignp_128B`。
- **L1243 EN**: Declares TableGen def record `V6_vavgh`.
  **L1243 CN**: 声明 TableGen def 记录 `V6_vavgh`。
- **L1244 EN**: Declares TableGen def record `V6_vavgh_128B`.
  **L1244 CN**: 声明 TableGen def 记录 `V6_vavgh_128B`。
- **L1245 EN**: Declares TableGen def record `V6_vavghrnd`.
  **L1245 CN**: 声明 TableGen def 记录 `V6_vavghrnd`。
- **L1246 EN**: Declares TableGen def record `V6_vavghrnd_128B`.
  **L1246 CN**: 声明 TableGen def 记录 `V6_vavghrnd_128B`。
- **L1247 EN**: Declares TableGen def record `V6_vavgub`.
  **L1247 CN**: 声明 TableGen def 记录 `V6_vavgub`。
- **L1248 EN**: Declares TableGen def record `V6_vavgub_128B`.
  **L1248 CN**: 声明 TableGen def 记录 `V6_vavgub_128B`。
- **L1249 EN**: Declares TableGen def record `V6_vavgubrnd`.
  **L1249 CN**: 声明 TableGen def 记录 `V6_vavgubrnd`。
- **L1250 EN**: Declares TableGen def record `V6_vavgubrnd_128B`.
  **L1250 CN**: 声明 TableGen def 记录 `V6_vavgubrnd_128B`。
- **L1251 EN**: Declares TableGen def record `V6_vavguh`.
  **L1251 CN**: 声明 TableGen def 记录 `V6_vavguh`。
- **L1252 EN**: Declares TableGen def record `V6_vavguh_128B`.
  **L1252 CN**: 声明 TableGen def 记录 `V6_vavguh_128B`。
- **L1253 EN**: Declares TableGen def record `V6_vavguhrnd`.
  **L1253 CN**: 声明 TableGen def 记录 `V6_vavguhrnd`。
- **L1254 EN**: Declares TableGen def record `V6_vavguhrnd_128B`.
  **L1254 CN**: 声明 TableGen def 记录 `V6_vavguhrnd_128B`。
- **L1255 EN**: Declares TableGen def record `V6_vavgw`.
  **L1255 CN**: 声明 TableGen def 记录 `V6_vavgw`。
- **L1256 EN**: Declares TableGen def record `V6_vavgw_128B`.
  **L1256 CN**: 声明 TableGen def 记录 `V6_vavgw_128B`。
- **L1257 EN**: Declares TableGen def record `V6_vavgwrnd`.
  **L1257 CN**: 声明 TableGen def 记录 `V6_vavgwrnd`。
- **L1258 EN**: Declares TableGen def record `V6_vavgwrnd_128B`.
  **L1258 CN**: 声明 TableGen def 记录 `V6_vavgwrnd_128B`。
- **L1259 EN**: Declares TableGen def record `V6_vcl0h`.
  **L1259 CN**: 声明 TableGen def 记录 `V6_vcl0h`。
- **L1260 EN**: Declares TableGen def record `V6_vcl0h_128B`.
  **L1260 CN**: 声明 TableGen def 记录 `V6_vcl0h_128B`。

### Lines 1261-1288

````tablegen
  def V6_vcl0w : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vcl0w_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vcombine : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vcombine_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vd0 : HexagonBuiltin<"_Vector<16, int>()">;
  def V6_vd0_128B : HexagonBuiltin<"_Vector<32, int>()">;
  def V6_vdealb : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vdealb_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vdealb4w : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vdealb4w_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vdealh : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vdealh_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vdealvdd : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>, _Vector<16, int>, int)">;
  def V6_vdealvdd_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vdelta : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vdelta_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vdmpybus : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, int)">;
  def V6_vdmpybus_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, int)">;
  def V6_vdmpybus_acc : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, int)">;
  def V6_vdmpybus_acc_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vdmpybus_dv : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, int)">;
  def V6_vdmpybus_dv_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, int)">;
  def V6_vdmpybus_dv_acc : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vdmpybus_dv_acc_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<64, int>, int)">;
  def V6_vdmpyhb : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, int)">;
  def V6_vdmpyhb_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, int)">;
  def V6_vdmpyhb_acc : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, int)">;
  def V6_vdmpyhb_acc_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
````
- **L1261 EN**: Declares TableGen def record `V6_vcl0w`.
  **L1261 CN**: 声明 TableGen def 记录 `V6_vcl0w`。
- **L1262 EN**: Declares TableGen def record `V6_vcl0w_128B`.
  **L1262 CN**: 声明 TableGen def 记录 `V6_vcl0w_128B`。
- **L1263 EN**: Declares TableGen def record `V6_vcombine`.
  **L1263 CN**: 声明 TableGen def 记录 `V6_vcombine`。
- **L1264 EN**: Declares TableGen def record `V6_vcombine_128B`.
  **L1264 CN**: 声明 TableGen def 记录 `V6_vcombine_128B`。
- **L1265 EN**: Declares TableGen def record `V6_vd0`.
  **L1265 CN**: 声明 TableGen def 记录 `V6_vd0`。
- **L1266 EN**: Declares TableGen def record `V6_vd0_128B`.
  **L1266 CN**: 声明 TableGen def 记录 `V6_vd0_128B`。
- **L1267 EN**: Declares TableGen def record `V6_vdealb`.
  **L1267 CN**: 声明 TableGen def 记录 `V6_vdealb`。
- **L1268 EN**: Declares TableGen def record `V6_vdealb_128B`.
  **L1268 CN**: 声明 TableGen def 记录 `V6_vdealb_128B`。
- **L1269 EN**: Declares TableGen def record `V6_vdealb4w`.
  **L1269 CN**: 声明 TableGen def 记录 `V6_vdealb4w`。
- **L1270 EN**: Declares TableGen def record `V6_vdealb4w_128B`.
  **L1270 CN**: 声明 TableGen def 记录 `V6_vdealb4w_128B`。
- **L1271 EN**: Declares TableGen def record `V6_vdealh`.
  **L1271 CN**: 声明 TableGen def 记录 `V6_vdealh`。
- **L1272 EN**: Declares TableGen def record `V6_vdealh_128B`.
  **L1272 CN**: 声明 TableGen def 记录 `V6_vdealh_128B`。
- **L1273 EN**: Declares TableGen def record `V6_vdealvdd`.
  **L1273 CN**: 声明 TableGen def 记录 `V6_vdealvdd`。
- **L1274 EN**: Declares TableGen def record `V6_vdealvdd_128B`.
  **L1274 CN**: 声明 TableGen def 记录 `V6_vdealvdd_128B`。
- **L1275 EN**: Declares TableGen def record `V6_vdelta`.
  **L1275 CN**: 声明 TableGen def 记录 `V6_vdelta`。
- **L1276 EN**: Declares TableGen def record `V6_vdelta_128B`.
  **L1276 CN**: 声明 TableGen def 记录 `V6_vdelta_128B`。
- **L1277 EN**: Declares TableGen def record `V6_vdmpybus`.
  **L1277 CN**: 声明 TableGen def 记录 `V6_vdmpybus`。
- **L1278 EN**: Declares TableGen def record `V6_vdmpybus_128B`.
  **L1278 CN**: 声明 TableGen def 记录 `V6_vdmpybus_128B`。
- **L1279 EN**: Declares TableGen def record `V6_vdmpybus_acc`.
  **L1279 CN**: 声明 TableGen def 记录 `V6_vdmpybus_acc`。
- **L1280 EN**: Declares TableGen def record `V6_vdmpybus_acc_128B`.
  **L1280 CN**: 声明 TableGen def 记录 `V6_vdmpybus_acc_128B`。
- **L1281 EN**: Declares TableGen def record `V6_vdmpybus_dv`.
  **L1281 CN**: 声明 TableGen def 记录 `V6_vdmpybus_dv`。
- **L1282 EN**: Declares TableGen def record `V6_vdmpybus_dv_128B`.
  **L1282 CN**: 声明 TableGen def 记录 `V6_vdmpybus_dv_128B`。
- **L1283 EN**: Declares TableGen def record `V6_vdmpybus_dv_acc`.
  **L1283 CN**: 声明 TableGen def 记录 `V6_vdmpybus_dv_acc`。
- **L1284 EN**: Declares TableGen def record `V6_vdmpybus_dv_acc_128B`.
  **L1284 CN**: 声明 TableGen def 记录 `V6_vdmpybus_dv_acc_128B`。
- **L1285 EN**: Declares TableGen def record `V6_vdmpyhb`.
  **L1285 CN**: 声明 TableGen def 记录 `V6_vdmpyhb`。
- **L1286 EN**: Declares TableGen def record `V6_vdmpyhb_128B`.
  **L1286 CN**: 声明 TableGen def 记录 `V6_vdmpyhb_128B`。
- **L1287 EN**: Declares TableGen def record `V6_vdmpyhb_acc`.
  **L1287 CN**: 声明 TableGen def 记录 `V6_vdmpyhb_acc`。
- **L1288 EN**: Declares TableGen def record `V6_vdmpyhb_acc_128B`.
  **L1288 CN**: 声明 TableGen def 记录 `V6_vdmpyhb_acc_128B`。

### Lines 1289-1316

````tablegen
  def V6_vdmpyhb_dv : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, int)">;
  def V6_vdmpyhb_dv_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, int)">;
  def V6_vdmpyhb_dv_acc : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vdmpyhb_dv_acc_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<64, int>, int)">;
  def V6_vdmpyhisat : HexagonBuiltin<"_Vector<16, int>(_Vector<32, int>, int)">;
  def V6_vdmpyhisat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<64, int>, int)">;
  def V6_vdmpyhisat_acc : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<32, int>, int)">;
  def V6_vdmpyhisat_acc_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<64, int>, int)">;
  def V6_vdmpyhsat : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, int)">;
  def V6_vdmpyhsat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, int)">;
  def V6_vdmpyhsat_acc : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, int)">;
  def V6_vdmpyhsat_acc_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vdmpyhsuisat : HexagonBuiltin<"_Vector<16, int>(_Vector<32, int>, int)">;
  def V6_vdmpyhsuisat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<64, int>, int)">;
  def V6_vdmpyhsuisat_acc : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<32, int>, int)">;
  def V6_vdmpyhsuisat_acc_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<64, int>, int)">;
  def V6_vdmpyhsusat : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, int)">;
  def V6_vdmpyhsusat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, int)">;
  def V6_vdmpyhsusat_acc : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, int)">;
  def V6_vdmpyhsusat_acc_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vdmpyhvsat : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vdmpyhvsat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vdmpyhvsat_acc : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vdmpyhvsat_acc_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vdsaduh : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, int)">;
  def V6_vdsaduh_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, int)">;
  def V6_vdsaduh_acc : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vdsaduh_acc_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<64, int>, int)">;
````
- **L1289 EN**: Declares TableGen def record `V6_vdmpyhb_dv`.
  **L1289 CN**: 声明 TableGen def 记录 `V6_vdmpyhb_dv`。
- **L1290 EN**: Declares TableGen def record `V6_vdmpyhb_dv_128B`.
  **L1290 CN**: 声明 TableGen def 记录 `V6_vdmpyhb_dv_128B`。
- **L1291 EN**: Declares TableGen def record `V6_vdmpyhb_dv_acc`.
  **L1291 CN**: 声明 TableGen def 记录 `V6_vdmpyhb_dv_acc`。
- **L1292 EN**: Declares TableGen def record `V6_vdmpyhb_dv_acc_128B`.
  **L1292 CN**: 声明 TableGen def 记录 `V6_vdmpyhb_dv_acc_128B`。
- **L1293 EN**: Declares TableGen def record `V6_vdmpyhisat`.
  **L1293 CN**: 声明 TableGen def 记录 `V6_vdmpyhisat`。
- **L1294 EN**: Declares TableGen def record `V6_vdmpyhisat_128B`.
  **L1294 CN**: 声明 TableGen def 记录 `V6_vdmpyhisat_128B`。
- **L1295 EN**: Declares TableGen def record `V6_vdmpyhisat_acc`.
  **L1295 CN**: 声明 TableGen def 记录 `V6_vdmpyhisat_acc`。
- **L1296 EN**: Declares TableGen def record `V6_vdmpyhisat_acc_128B`.
  **L1296 CN**: 声明 TableGen def 记录 `V6_vdmpyhisat_acc_128B`。
- **L1297 EN**: Declares TableGen def record `V6_vdmpyhsat`.
  **L1297 CN**: 声明 TableGen def 记录 `V6_vdmpyhsat`。
- **L1298 EN**: Declares TableGen def record `V6_vdmpyhsat_128B`.
  **L1298 CN**: 声明 TableGen def 记录 `V6_vdmpyhsat_128B`。
- **L1299 EN**: Declares TableGen def record `V6_vdmpyhsat_acc`.
  **L1299 CN**: 声明 TableGen def 记录 `V6_vdmpyhsat_acc`。
- **L1300 EN**: Declares TableGen def record `V6_vdmpyhsat_acc_128B`.
  **L1300 CN**: 声明 TableGen def 记录 `V6_vdmpyhsat_acc_128B`。
- **L1301 EN**: Declares TableGen def record `V6_vdmpyhsuisat`.
  **L1301 CN**: 声明 TableGen def 记录 `V6_vdmpyhsuisat`。
- **L1302 EN**: Declares TableGen def record `V6_vdmpyhsuisat_128B`.
  **L1302 CN**: 声明 TableGen def 记录 `V6_vdmpyhsuisat_128B`。
- **L1303 EN**: Declares TableGen def record `V6_vdmpyhsuisat_acc`.
  **L1303 CN**: 声明 TableGen def 记录 `V6_vdmpyhsuisat_acc`。
- **L1304 EN**: Declares TableGen def record `V6_vdmpyhsuisat_acc_128B`.
  **L1304 CN**: 声明 TableGen def 记录 `V6_vdmpyhsuisat_acc_128B`。
- **L1305 EN**: Declares TableGen def record `V6_vdmpyhsusat`.
  **L1305 CN**: 声明 TableGen def 记录 `V6_vdmpyhsusat`。
- **L1306 EN**: Declares TableGen def record `V6_vdmpyhsusat_128B`.
  **L1306 CN**: 声明 TableGen def 记录 `V6_vdmpyhsusat_128B`。
- **L1307 EN**: Declares TableGen def record `V6_vdmpyhsusat_acc`.
  **L1307 CN**: 声明 TableGen def 记录 `V6_vdmpyhsusat_acc`。
- **L1308 EN**: Declares TableGen def record `V6_vdmpyhsusat_acc_128B`.
  **L1308 CN**: 声明 TableGen def 记录 `V6_vdmpyhsusat_acc_128B`。
- **L1309 EN**: Declares TableGen def record `V6_vdmpyhvsat`.
  **L1309 CN**: 声明 TableGen def 记录 `V6_vdmpyhvsat`。
- **L1310 EN**: Declares TableGen def record `V6_vdmpyhvsat_128B`.
  **L1310 CN**: 声明 TableGen def 记录 `V6_vdmpyhvsat_128B`。
- **L1311 EN**: Declares TableGen def record `V6_vdmpyhvsat_acc`.
  **L1311 CN**: 声明 TableGen def 记录 `V6_vdmpyhvsat_acc`。
- **L1312 EN**: Declares TableGen def record `V6_vdmpyhvsat_acc_128B`.
  **L1312 CN**: 声明 TableGen def 记录 `V6_vdmpyhvsat_acc_128B`。
- **L1313 EN**: Declares TableGen def record `V6_vdsaduh`.
  **L1313 CN**: 声明 TableGen def 记录 `V6_vdsaduh`。
- **L1314 EN**: Declares TableGen def record `V6_vdsaduh_128B`.
  **L1314 CN**: 声明 TableGen def 记录 `V6_vdsaduh_128B`。
- **L1315 EN**: Declares TableGen def record `V6_vdsaduh_acc`.
  **L1315 CN**: 声明 TableGen def 记录 `V6_vdsaduh_acc`。
- **L1316 EN**: Declares TableGen def record `V6_vdsaduh_acc_128B`.
  **L1316 CN**: 声明 TableGen def 记录 `V6_vdsaduh_acc_128B`。

### Lines 1317-1344

````tablegen
  def V6_veqb : HexagonBuiltin<"_Vector<64, bool>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_veqb_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_veqb_and : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_veqb_and_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_veqb_or : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_veqb_or_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_veqb_xor : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_veqb_xor_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_veqh : HexagonBuiltin<"_Vector<64, bool>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_veqh_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_veqh_and : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_veqh_and_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_veqh_or : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_veqh_or_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_veqh_xor : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_veqh_xor_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_veqw : HexagonBuiltin<"_Vector<64, bool>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_veqw_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_veqw_and : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_veqw_and_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_veqw_or : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_veqw_or_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_veqw_xor : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_veqw_xor_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vgtb : HexagonBuiltin<"_Vector<64, bool>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vgtb_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vgtb_and : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vgtb_and_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
````
- **L1317 EN**: Declares TableGen def record `V6_veqb`.
  **L1317 CN**: 声明 TableGen def 记录 `V6_veqb`。
- **L1318 EN**: Declares TableGen def record `V6_veqb_128B`.
  **L1318 CN**: 声明 TableGen def 记录 `V6_veqb_128B`。
- **L1319 EN**: Declares TableGen def record `V6_veqb_and`.
  **L1319 CN**: 声明 TableGen def 记录 `V6_veqb_and`。
- **L1320 EN**: Declares TableGen def record `V6_veqb_and_128B`.
  **L1320 CN**: 声明 TableGen def 记录 `V6_veqb_and_128B`。
- **L1321 EN**: Declares TableGen def record `V6_veqb_or`.
  **L1321 CN**: 声明 TableGen def 记录 `V6_veqb_or`。
- **L1322 EN**: Declares TableGen def record `V6_veqb_or_128B`.
  **L1322 CN**: 声明 TableGen def 记录 `V6_veqb_or_128B`。
- **L1323 EN**: Declares TableGen def record `V6_veqb_xor`.
  **L1323 CN**: 声明 TableGen def 记录 `V6_veqb_xor`。
- **L1324 EN**: Declares TableGen def record `V6_veqb_xor_128B`.
  **L1324 CN**: 声明 TableGen def 记录 `V6_veqb_xor_128B`。
- **L1325 EN**: Declares TableGen def record `V6_veqh`.
  **L1325 CN**: 声明 TableGen def 记录 `V6_veqh`。
- **L1326 EN**: Declares TableGen def record `V6_veqh_128B`.
  **L1326 CN**: 声明 TableGen def 记录 `V6_veqh_128B`。
- **L1327 EN**: Declares TableGen def record `V6_veqh_and`.
  **L1327 CN**: 声明 TableGen def 记录 `V6_veqh_and`。
- **L1328 EN**: Declares TableGen def record `V6_veqh_and_128B`.
  **L1328 CN**: 声明 TableGen def 记录 `V6_veqh_and_128B`。
- **L1329 EN**: Declares TableGen def record `V6_veqh_or`.
  **L1329 CN**: 声明 TableGen def 记录 `V6_veqh_or`。
- **L1330 EN**: Declares TableGen def record `V6_veqh_or_128B`.
  **L1330 CN**: 声明 TableGen def 记录 `V6_veqh_or_128B`。
- **L1331 EN**: Declares TableGen def record `V6_veqh_xor`.
  **L1331 CN**: 声明 TableGen def 记录 `V6_veqh_xor`。
- **L1332 EN**: Declares TableGen def record `V6_veqh_xor_128B`.
  **L1332 CN**: 声明 TableGen def 记录 `V6_veqh_xor_128B`。
- **L1333 EN**: Declares TableGen def record `V6_veqw`.
  **L1333 CN**: 声明 TableGen def 记录 `V6_veqw`。
- **L1334 EN**: Declares TableGen def record `V6_veqw_128B`.
  **L1334 CN**: 声明 TableGen def 记录 `V6_veqw_128B`。
- **L1335 EN**: Declares TableGen def record `V6_veqw_and`.
  **L1335 CN**: 声明 TableGen def 记录 `V6_veqw_and`。
- **L1336 EN**: Declares TableGen def record `V6_veqw_and_128B`.
  **L1336 CN**: 声明 TableGen def 记录 `V6_veqw_and_128B`。
- **L1337 EN**: Declares TableGen def record `V6_veqw_or`.
  **L1337 CN**: 声明 TableGen def 记录 `V6_veqw_or`。
- **L1338 EN**: Declares TableGen def record `V6_veqw_or_128B`.
  **L1338 CN**: 声明 TableGen def 记录 `V6_veqw_or_128B`。
- **L1339 EN**: Declares TableGen def record `V6_veqw_xor`.
  **L1339 CN**: 声明 TableGen def 记录 `V6_veqw_xor`。
- **L1340 EN**: Declares TableGen def record `V6_veqw_xor_128B`.
  **L1340 CN**: 声明 TableGen def 记录 `V6_veqw_xor_128B`。
- **L1341 EN**: Declares TableGen def record `V6_vgtb`.
  **L1341 CN**: 声明 TableGen def 记录 `V6_vgtb`。
- **L1342 EN**: Declares TableGen def record `V6_vgtb_128B`.
  **L1342 CN**: 声明 TableGen def 记录 `V6_vgtb_128B`。
- **L1343 EN**: Declares TableGen def record `V6_vgtb_and`.
  **L1343 CN**: 声明 TableGen def 记录 `V6_vgtb_and`。
- **L1344 EN**: Declares TableGen def record `V6_vgtb_and_128B`.
  **L1344 CN**: 声明 TableGen def 记录 `V6_vgtb_and_128B`。

### Lines 1345-1372

````tablegen
  def V6_vgtb_or : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vgtb_or_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vgtb_xor : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vgtb_xor_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vgth : HexagonBuiltin<"_Vector<64, bool>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vgth_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vgth_and : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vgth_and_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vgth_or : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vgth_or_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vgth_xor : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vgth_xor_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vgtub : HexagonBuiltin<"_Vector<64, bool>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vgtub_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vgtub_and : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vgtub_and_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vgtub_or : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vgtub_or_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vgtub_xor : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vgtub_xor_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vgtuh : HexagonBuiltin<"_Vector<64, bool>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vgtuh_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vgtuh_and : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vgtuh_and_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vgtuh_or : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vgtuh_or_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vgtuh_xor : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vgtuh_xor_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
````
- **L1345 EN**: Declares TableGen def record `V6_vgtb_or`.
  **L1345 CN**: 声明 TableGen def 记录 `V6_vgtb_or`。
- **L1346 EN**: Declares TableGen def record `V6_vgtb_or_128B`.
  **L1346 CN**: 声明 TableGen def 记录 `V6_vgtb_or_128B`。
- **L1347 EN**: Declares TableGen def record `V6_vgtb_xor`.
  **L1347 CN**: 声明 TableGen def 记录 `V6_vgtb_xor`。
- **L1348 EN**: Declares TableGen def record `V6_vgtb_xor_128B`.
  **L1348 CN**: 声明 TableGen def 记录 `V6_vgtb_xor_128B`。
- **L1349 EN**: Declares TableGen def record `V6_vgth`.
  **L1349 CN**: 声明 TableGen def 记录 `V6_vgth`。
- **L1350 EN**: Declares TableGen def record `V6_vgth_128B`.
  **L1350 CN**: 声明 TableGen def 记录 `V6_vgth_128B`。
- **L1351 EN**: Declares TableGen def record `V6_vgth_and`.
  **L1351 CN**: 声明 TableGen def 记录 `V6_vgth_and`。
- **L1352 EN**: Declares TableGen def record `V6_vgth_and_128B`.
  **L1352 CN**: 声明 TableGen def 记录 `V6_vgth_and_128B`。
- **L1353 EN**: Declares TableGen def record `V6_vgth_or`.
  **L1353 CN**: 声明 TableGen def 记录 `V6_vgth_or`。
- **L1354 EN**: Declares TableGen def record `V6_vgth_or_128B`.
  **L1354 CN**: 声明 TableGen def 记录 `V6_vgth_or_128B`。
- **L1355 EN**: Declares TableGen def record `V6_vgth_xor`.
  **L1355 CN**: 声明 TableGen def 记录 `V6_vgth_xor`。
- **L1356 EN**: Declares TableGen def record `V6_vgth_xor_128B`.
  **L1356 CN**: 声明 TableGen def 记录 `V6_vgth_xor_128B`。
- **L1357 EN**: Declares TableGen def record `V6_vgtub`.
  **L1357 CN**: 声明 TableGen def 记录 `V6_vgtub`。
- **L1358 EN**: Declares TableGen def record `V6_vgtub_128B`.
  **L1358 CN**: 声明 TableGen def 记录 `V6_vgtub_128B`。
- **L1359 EN**: Declares TableGen def record `V6_vgtub_and`.
  **L1359 CN**: 声明 TableGen def 记录 `V6_vgtub_and`。
- **L1360 EN**: Declares TableGen def record `V6_vgtub_and_128B`.
  **L1360 CN**: 声明 TableGen def 记录 `V6_vgtub_and_128B`。
- **L1361 EN**: Declares TableGen def record `V6_vgtub_or`.
  **L1361 CN**: 声明 TableGen def 记录 `V6_vgtub_or`。
- **L1362 EN**: Declares TableGen def record `V6_vgtub_or_128B`.
  **L1362 CN**: 声明 TableGen def 记录 `V6_vgtub_or_128B`。
- **L1363 EN**: Declares TableGen def record `V6_vgtub_xor`.
  **L1363 CN**: 声明 TableGen def 记录 `V6_vgtub_xor`。
- **L1364 EN**: Declares TableGen def record `V6_vgtub_xor_128B`.
  **L1364 CN**: 声明 TableGen def 记录 `V6_vgtub_xor_128B`。
- **L1365 EN**: Declares TableGen def record `V6_vgtuh`.
  **L1365 CN**: 声明 TableGen def 记录 `V6_vgtuh`。
- **L1366 EN**: Declares TableGen def record `V6_vgtuh_128B`.
  **L1366 CN**: 声明 TableGen def 记录 `V6_vgtuh_128B`。
- **L1367 EN**: Declares TableGen def record `V6_vgtuh_and`.
  **L1367 CN**: 声明 TableGen def 记录 `V6_vgtuh_and`。
- **L1368 EN**: Declares TableGen def record `V6_vgtuh_and_128B`.
  **L1368 CN**: 声明 TableGen def 记录 `V6_vgtuh_and_128B`。
- **L1369 EN**: Declares TableGen def record `V6_vgtuh_or`.
  **L1369 CN**: 声明 TableGen def 记录 `V6_vgtuh_or`。
- **L1370 EN**: Declares TableGen def record `V6_vgtuh_or_128B`.
  **L1370 CN**: 声明 TableGen def 记录 `V6_vgtuh_or_128B`。
- **L1371 EN**: Declares TableGen def record `V6_vgtuh_xor`.
  **L1371 CN**: 声明 TableGen def 记录 `V6_vgtuh_xor`。
- **L1372 EN**: Declares TableGen def record `V6_vgtuh_xor_128B`.
  **L1372 CN**: 声明 TableGen def 记录 `V6_vgtuh_xor_128B`。

### Lines 1373-1400

````tablegen
  def V6_vgtuw : HexagonBuiltin<"_Vector<64, bool>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vgtuw_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vgtuw_and : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vgtuw_and_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vgtuw_or : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vgtuw_or_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vgtuw_xor : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vgtuw_xor_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vgtw : HexagonBuiltin<"_Vector<64, bool>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vgtw_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vgtw_and : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vgtw_and_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vgtw_or : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vgtw_or_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vgtw_xor : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vgtw_xor_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vinsertwr : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, int)">;
  def V6_vinsertwr_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, int)">;
  def V6_vlalignb : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, int)">;
  def V6_vlalignb_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vlalignbi : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, unsigned _Constant int)">;
  def V6_vlalignbi_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, unsigned _Constant int)">;
  def V6_vlsrh : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, int)">;
  def V6_vlsrh_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, int)">;
  def V6_vlsrhv : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vlsrhv_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vlsrw : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, int)">;
  def V6_vlsrw_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, int)">;
````
- **L1373 EN**: Declares TableGen def record `V6_vgtuw`.
  **L1373 CN**: 声明 TableGen def 记录 `V6_vgtuw`。
- **L1374 EN**: Declares TableGen def record `V6_vgtuw_128B`.
  **L1374 CN**: 声明 TableGen def 记录 `V6_vgtuw_128B`。
- **L1375 EN**: Declares TableGen def record `V6_vgtuw_and`.
  **L1375 CN**: 声明 TableGen def 记录 `V6_vgtuw_and`。
- **L1376 EN**: Declares TableGen def record `V6_vgtuw_and_128B`.
  **L1376 CN**: 声明 TableGen def 记录 `V6_vgtuw_and_128B`。
- **L1377 EN**: Declares TableGen def record `V6_vgtuw_or`.
  **L1377 CN**: 声明 TableGen def 记录 `V6_vgtuw_or`。
- **L1378 EN**: Declares TableGen def record `V6_vgtuw_or_128B`.
  **L1378 CN**: 声明 TableGen def 记录 `V6_vgtuw_or_128B`。
- **L1379 EN**: Declares TableGen def record `V6_vgtuw_xor`.
  **L1379 CN**: 声明 TableGen def 记录 `V6_vgtuw_xor`。
- **L1380 EN**: Declares TableGen def record `V6_vgtuw_xor_128B`.
  **L1380 CN**: 声明 TableGen def 记录 `V6_vgtuw_xor_128B`。
- **L1381 EN**: Declares TableGen def record `V6_vgtw`.
  **L1381 CN**: 声明 TableGen def 记录 `V6_vgtw`。
- **L1382 EN**: Declares TableGen def record `V6_vgtw_128B`.
  **L1382 CN**: 声明 TableGen def 记录 `V6_vgtw_128B`。
- **L1383 EN**: Declares TableGen def record `V6_vgtw_and`.
  **L1383 CN**: 声明 TableGen def 记录 `V6_vgtw_and`。
- **L1384 EN**: Declares TableGen def record `V6_vgtw_and_128B`.
  **L1384 CN**: 声明 TableGen def 记录 `V6_vgtw_and_128B`。
- **L1385 EN**: Declares TableGen def record `V6_vgtw_or`.
  **L1385 CN**: 声明 TableGen def 记录 `V6_vgtw_or`。
- **L1386 EN**: Declares TableGen def record `V6_vgtw_or_128B`.
  **L1386 CN**: 声明 TableGen def 记录 `V6_vgtw_or_128B`。
- **L1387 EN**: Declares TableGen def record `V6_vgtw_xor`.
  **L1387 CN**: 声明 TableGen def 记录 `V6_vgtw_xor`。
- **L1388 EN**: Declares TableGen def record `V6_vgtw_xor_128B`.
  **L1388 CN**: 声明 TableGen def 记录 `V6_vgtw_xor_128B`。
- **L1389 EN**: Declares TableGen def record `V6_vinsertwr`.
  **L1389 CN**: 声明 TableGen def 记录 `V6_vinsertwr`。
- **L1390 EN**: Declares TableGen def record `V6_vinsertwr_128B`.
  **L1390 CN**: 声明 TableGen def 记录 `V6_vinsertwr_128B`。
- **L1391 EN**: Declares TableGen def record `V6_vlalignb`.
  **L1391 CN**: 声明 TableGen def 记录 `V6_vlalignb`。
- **L1392 EN**: Declares TableGen def record `V6_vlalignb_128B`.
  **L1392 CN**: 声明 TableGen def 记录 `V6_vlalignb_128B`。
- **L1393 EN**: Declares TableGen def record `V6_vlalignbi`.
  **L1393 CN**: 声明 TableGen def 记录 `V6_vlalignbi`。
- **L1394 EN**: Declares TableGen def record `V6_vlalignbi_128B`.
  **L1394 CN**: 声明 TableGen def 记录 `V6_vlalignbi_128B`。
- **L1395 EN**: Declares TableGen def record `V6_vlsrh`.
  **L1395 CN**: 声明 TableGen def 记录 `V6_vlsrh`。
- **L1396 EN**: Declares TableGen def record `V6_vlsrh_128B`.
  **L1396 CN**: 声明 TableGen def 记录 `V6_vlsrh_128B`。
- **L1397 EN**: Declares TableGen def record `V6_vlsrhv`.
  **L1397 CN**: 声明 TableGen def 记录 `V6_vlsrhv`。
- **L1398 EN**: Declares TableGen def record `V6_vlsrhv_128B`.
  **L1398 CN**: 声明 TableGen def 记录 `V6_vlsrhv_128B`。
- **L1399 EN**: Declares TableGen def record `V6_vlsrw`.
  **L1399 CN**: 声明 TableGen def 记录 `V6_vlsrw`。
- **L1400 EN**: Declares TableGen def record `V6_vlsrw_128B`.
  **L1400 CN**: 声明 TableGen def 记录 `V6_vlsrw_128B`。

### Lines 1401-1428

````tablegen
  def V6_vlsrwv : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vlsrwv_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vlutvvb : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, int)">;
  def V6_vlutvvb_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vlutvvb_oracc : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, _Vector<16, int>, int)">;
  def V6_vlutvvb_oracc_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vlutvwh : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>, _Vector<16, int>, int)">;
  def V6_vlutvwh_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vlutvwh_oracc : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<16, int>, _Vector<16, int>, int)">;
  def V6_vlutvwh_oracc_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vmaxh : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vmaxh_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vmaxub : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vmaxub_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vmaxuh : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vmaxuh_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vmaxw : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vmaxw_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vminh : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vminh_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vminub : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vminub_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vminuh : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vminuh_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vminw : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vminw_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpabus : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, int)">;
  def V6_vmpabus_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, int)">;
````
- **L1401 EN**: Declares TableGen def record `V6_vlsrwv`.
  **L1401 CN**: 声明 TableGen def 记录 `V6_vlsrwv`。
- **L1402 EN**: Declares TableGen def record `V6_vlsrwv_128B`.
  **L1402 CN**: 声明 TableGen def 记录 `V6_vlsrwv_128B`。
- **L1403 EN**: Declares TableGen def record `V6_vlutvvb`.
  **L1403 CN**: 声明 TableGen def 记录 `V6_vlutvvb`。
- **L1404 EN**: Declares TableGen def record `V6_vlutvvb_128B`.
  **L1404 CN**: 声明 TableGen def 记录 `V6_vlutvvb_128B`。
- **L1405 EN**: Declares TableGen def record `V6_vlutvvb_oracc`.
  **L1405 CN**: 声明 TableGen def 记录 `V6_vlutvvb_oracc`。
- **L1406 EN**: Declares TableGen def record `V6_vlutvvb_oracc_128B`.
  **L1406 CN**: 声明 TableGen def 记录 `V6_vlutvvb_oracc_128B`。
- **L1407 EN**: Declares TableGen def record `V6_vlutvwh`.
  **L1407 CN**: 声明 TableGen def 记录 `V6_vlutvwh`。
- **L1408 EN**: Declares TableGen def record `V6_vlutvwh_128B`.
  **L1408 CN**: 声明 TableGen def 记录 `V6_vlutvwh_128B`。
- **L1409 EN**: Declares TableGen def record `V6_vlutvwh_oracc`.
  **L1409 CN**: 声明 TableGen def 记录 `V6_vlutvwh_oracc`。
- **L1410 EN**: Declares TableGen def record `V6_vlutvwh_oracc_128B`.
  **L1410 CN**: 声明 TableGen def 记录 `V6_vlutvwh_oracc_128B`。
- **L1411 EN**: Declares TableGen def record `V6_vmaxh`.
  **L1411 CN**: 声明 TableGen def 记录 `V6_vmaxh`。
- **L1412 EN**: Declares TableGen def record `V6_vmaxh_128B`.
  **L1412 CN**: 声明 TableGen def 记录 `V6_vmaxh_128B`。
- **L1413 EN**: Declares TableGen def record `V6_vmaxub`.
  **L1413 CN**: 声明 TableGen def 记录 `V6_vmaxub`。
- **L1414 EN**: Declares TableGen def record `V6_vmaxub_128B`.
  **L1414 CN**: 声明 TableGen def 记录 `V6_vmaxub_128B`。
- **L1415 EN**: Declares TableGen def record `V6_vmaxuh`.
  **L1415 CN**: 声明 TableGen def 记录 `V6_vmaxuh`。
- **L1416 EN**: Declares TableGen def record `V6_vmaxuh_128B`.
  **L1416 CN**: 声明 TableGen def 记录 `V6_vmaxuh_128B`。
- **L1417 EN**: Declares TableGen def record `V6_vmaxw`.
  **L1417 CN**: 声明 TableGen def 记录 `V6_vmaxw`。
- **L1418 EN**: Declares TableGen def record `V6_vmaxw_128B`.
  **L1418 CN**: 声明 TableGen def 记录 `V6_vmaxw_128B`。
- **L1419 EN**: Declares TableGen def record `V6_vminh`.
  **L1419 CN**: 声明 TableGen def 记录 `V6_vminh`。
- **L1420 EN**: Declares TableGen def record `V6_vminh_128B`.
  **L1420 CN**: 声明 TableGen def 记录 `V6_vminh_128B`。
- **L1421 EN**: Declares TableGen def record `V6_vminub`.
  **L1421 CN**: 声明 TableGen def 记录 `V6_vminub`。
- **L1422 EN**: Declares TableGen def record `V6_vminub_128B`.
  **L1422 CN**: 声明 TableGen def 记录 `V6_vminub_128B`。
- **L1423 EN**: Declares TableGen def record `V6_vminuh`.
  **L1423 CN**: 声明 TableGen def 记录 `V6_vminuh`。
- **L1424 EN**: Declares TableGen def record `V6_vminuh_128B`.
  **L1424 CN**: 声明 TableGen def 记录 `V6_vminuh_128B`。
- **L1425 EN**: Declares TableGen def record `V6_vminw`.
  **L1425 CN**: 声明 TableGen def 记录 `V6_vminw`。
- **L1426 EN**: Declares TableGen def record `V6_vminw_128B`.
  **L1426 CN**: 声明 TableGen def 记录 `V6_vminw_128B`。
- **L1427 EN**: Declares TableGen def record `V6_vmpabus`.
  **L1427 CN**: 声明 TableGen def 记录 `V6_vmpabus`。
- **L1428 EN**: Declares TableGen def record `V6_vmpabus_128B`.
  **L1428 CN**: 声明 TableGen def 记录 `V6_vmpabus_128B`。

### Lines 1429-1456

````tablegen
  def V6_vmpabus_acc : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vmpabus_acc_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<64, int>, int)">;
  def V6_vmpabusv : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpabusv_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<64, int>)">;
  def V6_vmpabuuv : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpabuuv_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<64, int>)">;
  def V6_vmpahb : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, int)">;
  def V6_vmpahb_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, int)">;
  def V6_vmpahb_acc : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vmpahb_acc_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<64, int>, int)">;
  def V6_vmpybus : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>, int)">;
  def V6_vmpybus_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>, int)">;
  def V6_vmpybus_acc : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<16, int>, int)">;
  def V6_vmpybus_acc_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<32, int>, int)">;
  def V6_vmpybusv : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpybusv_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpybusv_acc : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpybusv_acc_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpybv : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpybv_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpybv_acc : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpybv_acc_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpyewuh : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpyewuh_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpyh : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>, int)">;
  def V6_vmpyh_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>, int)">;
  def V6_vmpyhsat_acc : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<16, int>, int)">;
  def V6_vmpyhsat_acc_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<32, int>, int)">;
````
- **L1429 EN**: Declares TableGen def record `V6_vmpabus_acc`.
  **L1429 CN**: 声明 TableGen def 记录 `V6_vmpabus_acc`。
- **L1430 EN**: Declares TableGen def record `V6_vmpabus_acc_128B`.
  **L1430 CN**: 声明 TableGen def 记录 `V6_vmpabus_acc_128B`。
- **L1431 EN**: Declares TableGen def record `V6_vmpabusv`.
  **L1431 CN**: 声明 TableGen def 记录 `V6_vmpabusv`。
- **L1432 EN**: Declares TableGen def record `V6_vmpabusv_128B`.
  **L1432 CN**: 声明 TableGen def 记录 `V6_vmpabusv_128B`。
- **L1433 EN**: Declares TableGen def record `V6_vmpabuuv`.
  **L1433 CN**: 声明 TableGen def 记录 `V6_vmpabuuv`。
- **L1434 EN**: Declares TableGen def record `V6_vmpabuuv_128B`.
  **L1434 CN**: 声明 TableGen def 记录 `V6_vmpabuuv_128B`。
- **L1435 EN**: Declares TableGen def record `V6_vmpahb`.
  **L1435 CN**: 声明 TableGen def 记录 `V6_vmpahb`。
- **L1436 EN**: Declares TableGen def record `V6_vmpahb_128B`.
  **L1436 CN**: 声明 TableGen def 记录 `V6_vmpahb_128B`。
- **L1437 EN**: Declares TableGen def record `V6_vmpahb_acc`.
  **L1437 CN**: 声明 TableGen def 记录 `V6_vmpahb_acc`。
- **L1438 EN**: Declares TableGen def record `V6_vmpahb_acc_128B`.
  **L1438 CN**: 声明 TableGen def 记录 `V6_vmpahb_acc_128B`。
- **L1439 EN**: Declares TableGen def record `V6_vmpybus`.
  **L1439 CN**: 声明 TableGen def 记录 `V6_vmpybus`。
- **L1440 EN**: Declares TableGen def record `V6_vmpybus_128B`.
  **L1440 CN**: 声明 TableGen def 记录 `V6_vmpybus_128B`。
- **L1441 EN**: Declares TableGen def record `V6_vmpybus_acc`.
  **L1441 CN**: 声明 TableGen def 记录 `V6_vmpybus_acc`。
- **L1442 EN**: Declares TableGen def record `V6_vmpybus_acc_128B`.
  **L1442 CN**: 声明 TableGen def 记录 `V6_vmpybus_acc_128B`。
- **L1443 EN**: Declares TableGen def record `V6_vmpybusv`.
  **L1443 CN**: 声明 TableGen def 记录 `V6_vmpybusv`。
- **L1444 EN**: Declares TableGen def record `V6_vmpybusv_128B`.
  **L1444 CN**: 声明 TableGen def 记录 `V6_vmpybusv_128B`。
- **L1445 EN**: Declares TableGen def record `V6_vmpybusv_acc`.
  **L1445 CN**: 声明 TableGen def 记录 `V6_vmpybusv_acc`。
- **L1446 EN**: Declares TableGen def record `V6_vmpybusv_acc_128B`.
  **L1446 CN**: 声明 TableGen def 记录 `V6_vmpybusv_acc_128B`。
- **L1447 EN**: Declares TableGen def record `V6_vmpybv`.
  **L1447 CN**: 声明 TableGen def 记录 `V6_vmpybv`。
- **L1448 EN**: Declares TableGen def record `V6_vmpybv_128B`.
  **L1448 CN**: 声明 TableGen def 记录 `V6_vmpybv_128B`。
- **L1449 EN**: Declares TableGen def record `V6_vmpybv_acc`.
  **L1449 CN**: 声明 TableGen def 记录 `V6_vmpybv_acc`。
- **L1450 EN**: Declares TableGen def record `V6_vmpybv_acc_128B`.
  **L1450 CN**: 声明 TableGen def 记录 `V6_vmpybv_acc_128B`。
- **L1451 EN**: Declares TableGen def record `V6_vmpyewuh`.
  **L1451 CN**: 声明 TableGen def 记录 `V6_vmpyewuh`。
- **L1452 EN**: Declares TableGen def record `V6_vmpyewuh_128B`.
  **L1452 CN**: 声明 TableGen def 记录 `V6_vmpyewuh_128B`。
- **L1453 EN**: Declares TableGen def record `V6_vmpyh`.
  **L1453 CN**: 声明 TableGen def 记录 `V6_vmpyh`。
- **L1454 EN**: Declares TableGen def record `V6_vmpyh_128B`.
  **L1454 CN**: 声明 TableGen def 记录 `V6_vmpyh_128B`。
- **L1455 EN**: Declares TableGen def record `V6_vmpyhsat_acc`.
  **L1455 CN**: 声明 TableGen def 记录 `V6_vmpyhsat_acc`。
- **L1456 EN**: Declares TableGen def record `V6_vmpyhsat_acc_128B`.
  **L1456 CN**: 声明 TableGen def 记录 `V6_vmpyhsat_acc_128B`。

### Lines 1457-1484

````tablegen
  def V6_vmpyhsrs : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, int)">;
  def V6_vmpyhsrs_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, int)">;
  def V6_vmpyhss : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, int)">;
  def V6_vmpyhss_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, int)">;
  def V6_vmpyhus : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpyhus_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpyhus_acc : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpyhus_acc_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpyhv : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpyhv_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpyhv_acc : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpyhv_acc_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpyhvsrs : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpyhvsrs_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpyieoh : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpyieoh_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpyiewh_acc : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpyiewh_acc_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpyiewuh : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpyiewuh_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpyiewuh_acc : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpyiewuh_acc_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpyih : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpyih_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpyih_acc : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpyih_acc_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpyihb : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, int)">;
  def V6_vmpyihb_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, int)">;
````
- **L1457 EN**: Declares TableGen def record `V6_vmpyhsrs`.
  **L1457 CN**: 声明 TableGen def 记录 `V6_vmpyhsrs`。
- **L1458 EN**: Declares TableGen def record `V6_vmpyhsrs_128B`.
  **L1458 CN**: 声明 TableGen def 记录 `V6_vmpyhsrs_128B`。
- **L1459 EN**: Declares TableGen def record `V6_vmpyhss`.
  **L1459 CN**: 声明 TableGen def 记录 `V6_vmpyhss`。
- **L1460 EN**: Declares TableGen def record `V6_vmpyhss_128B`.
  **L1460 CN**: 声明 TableGen def 记录 `V6_vmpyhss_128B`。
- **L1461 EN**: Declares TableGen def record `V6_vmpyhus`.
  **L1461 CN**: 声明 TableGen def 记录 `V6_vmpyhus`。
- **L1462 EN**: Declares TableGen def record `V6_vmpyhus_128B`.
  **L1462 CN**: 声明 TableGen def 记录 `V6_vmpyhus_128B`。
- **L1463 EN**: Declares TableGen def record `V6_vmpyhus_acc`.
  **L1463 CN**: 声明 TableGen def 记录 `V6_vmpyhus_acc`。
- **L1464 EN**: Declares TableGen def record `V6_vmpyhus_acc_128B`.
  **L1464 CN**: 声明 TableGen def 记录 `V6_vmpyhus_acc_128B`。
- **L1465 EN**: Declares TableGen def record `V6_vmpyhv`.
  **L1465 CN**: 声明 TableGen def 记录 `V6_vmpyhv`。
- **L1466 EN**: Declares TableGen def record `V6_vmpyhv_128B`.
  **L1466 CN**: 声明 TableGen def 记录 `V6_vmpyhv_128B`。
- **L1467 EN**: Declares TableGen def record `V6_vmpyhv_acc`.
  **L1467 CN**: 声明 TableGen def 记录 `V6_vmpyhv_acc`。
- **L1468 EN**: Declares TableGen def record `V6_vmpyhv_acc_128B`.
  **L1468 CN**: 声明 TableGen def 记录 `V6_vmpyhv_acc_128B`。
- **L1469 EN**: Declares TableGen def record `V6_vmpyhvsrs`.
  **L1469 CN**: 声明 TableGen def 记录 `V6_vmpyhvsrs`。
- **L1470 EN**: Declares TableGen def record `V6_vmpyhvsrs_128B`.
  **L1470 CN**: 声明 TableGen def 记录 `V6_vmpyhvsrs_128B`。
- **L1471 EN**: Declares TableGen def record `V6_vmpyieoh`.
  **L1471 CN**: 声明 TableGen def 记录 `V6_vmpyieoh`。
- **L1472 EN**: Declares TableGen def record `V6_vmpyieoh_128B`.
  **L1472 CN**: 声明 TableGen def 记录 `V6_vmpyieoh_128B`。
- **L1473 EN**: Declares TableGen def record `V6_vmpyiewh_acc`.
  **L1473 CN**: 声明 TableGen def 记录 `V6_vmpyiewh_acc`。
- **L1474 EN**: Declares TableGen def record `V6_vmpyiewh_acc_128B`.
  **L1474 CN**: 声明 TableGen def 记录 `V6_vmpyiewh_acc_128B`。
- **L1475 EN**: Declares TableGen def record `V6_vmpyiewuh`.
  **L1475 CN**: 声明 TableGen def 记录 `V6_vmpyiewuh`。
- **L1476 EN**: Declares TableGen def record `V6_vmpyiewuh_128B`.
  **L1476 CN**: 声明 TableGen def 记录 `V6_vmpyiewuh_128B`。
- **L1477 EN**: Declares TableGen def record `V6_vmpyiewuh_acc`.
  **L1477 CN**: 声明 TableGen def 记录 `V6_vmpyiewuh_acc`。
- **L1478 EN**: Declares TableGen def record `V6_vmpyiewuh_acc_128B`.
  **L1478 CN**: 声明 TableGen def 记录 `V6_vmpyiewuh_acc_128B`。
- **L1479 EN**: Declares TableGen def record `V6_vmpyih`.
  **L1479 CN**: 声明 TableGen def 记录 `V6_vmpyih`。
- **L1480 EN**: Declares TableGen def record `V6_vmpyih_128B`.
  **L1480 CN**: 声明 TableGen def 记录 `V6_vmpyih_128B`。
- **L1481 EN**: Declares TableGen def record `V6_vmpyih_acc`.
  **L1481 CN**: 声明 TableGen def 记录 `V6_vmpyih_acc`。
- **L1482 EN**: Declares TableGen def record `V6_vmpyih_acc_128B`.
  **L1482 CN**: 声明 TableGen def 记录 `V6_vmpyih_acc_128B`。
- **L1483 EN**: Declares TableGen def record `V6_vmpyihb`.
  **L1483 CN**: 声明 TableGen def 记录 `V6_vmpyihb`。
- **L1484 EN**: Declares TableGen def record `V6_vmpyihb_128B`.
  **L1484 CN**: 声明 TableGen def 记录 `V6_vmpyihb_128B`。

### Lines 1485-1512

````tablegen
  def V6_vmpyihb_acc : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, int)">;
  def V6_vmpyihb_acc_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vmpyiowh : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpyiowh_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpyiwb : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, int)">;
  def V6_vmpyiwb_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, int)">;
  def V6_vmpyiwb_acc : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, int)">;
  def V6_vmpyiwb_acc_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vmpyiwh : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, int)">;
  def V6_vmpyiwh_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, int)">;
  def V6_vmpyiwh_acc : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, int)">;
  def V6_vmpyiwh_acc_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vmpyowh : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpyowh_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpyowh_rnd : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpyowh_rnd_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpyowh_rnd_sacc : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpyowh_rnd_sacc_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpyowh_sacc : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpyowh_sacc_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpyub : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>, int)">;
  def V6_vmpyub_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>, int)">;
  def V6_vmpyub_acc : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<16, int>, int)">;
  def V6_vmpyub_acc_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<32, int>, int)">;
  def V6_vmpyubv : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpyubv_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpyubv_acc : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpyubv_acc_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<32, int>, _Vector<32, int>)">;
````
- **L1485 EN**: Declares TableGen def record `V6_vmpyihb_acc`.
  **L1485 CN**: 声明 TableGen def 记录 `V6_vmpyihb_acc`。
- **L1486 EN**: Declares TableGen def record `V6_vmpyihb_acc_128B`.
  **L1486 CN**: 声明 TableGen def 记录 `V6_vmpyihb_acc_128B`。
- **L1487 EN**: Declares TableGen def record `V6_vmpyiowh`.
  **L1487 CN**: 声明 TableGen def 记录 `V6_vmpyiowh`。
- **L1488 EN**: Declares TableGen def record `V6_vmpyiowh_128B`.
  **L1488 CN**: 声明 TableGen def 记录 `V6_vmpyiowh_128B`。
- **L1489 EN**: Declares TableGen def record `V6_vmpyiwb`.
  **L1489 CN**: 声明 TableGen def 记录 `V6_vmpyiwb`。
- **L1490 EN**: Declares TableGen def record `V6_vmpyiwb_128B`.
  **L1490 CN**: 声明 TableGen def 记录 `V6_vmpyiwb_128B`。
- **L1491 EN**: Declares TableGen def record `V6_vmpyiwb_acc`.
  **L1491 CN**: 声明 TableGen def 记录 `V6_vmpyiwb_acc`。
- **L1492 EN**: Declares TableGen def record `V6_vmpyiwb_acc_128B`.
  **L1492 CN**: 声明 TableGen def 记录 `V6_vmpyiwb_acc_128B`。
- **L1493 EN**: Declares TableGen def record `V6_vmpyiwh`.
  **L1493 CN**: 声明 TableGen def 记录 `V6_vmpyiwh`。
- **L1494 EN**: Declares TableGen def record `V6_vmpyiwh_128B`.
  **L1494 CN**: 声明 TableGen def 记录 `V6_vmpyiwh_128B`。
- **L1495 EN**: Declares TableGen def record `V6_vmpyiwh_acc`.
  **L1495 CN**: 声明 TableGen def 记录 `V6_vmpyiwh_acc`。
- **L1496 EN**: Declares TableGen def record `V6_vmpyiwh_acc_128B`.
  **L1496 CN**: 声明 TableGen def 记录 `V6_vmpyiwh_acc_128B`。
- **L1497 EN**: Declares TableGen def record `V6_vmpyowh`.
  **L1497 CN**: 声明 TableGen def 记录 `V6_vmpyowh`。
- **L1498 EN**: Declares TableGen def record `V6_vmpyowh_128B`.
  **L1498 CN**: 声明 TableGen def 记录 `V6_vmpyowh_128B`。
- **L1499 EN**: Declares TableGen def record `V6_vmpyowh_rnd`.
  **L1499 CN**: 声明 TableGen def 记录 `V6_vmpyowh_rnd`。
- **L1500 EN**: Declares TableGen def record `V6_vmpyowh_rnd_128B`.
  **L1500 CN**: 声明 TableGen def 记录 `V6_vmpyowh_rnd_128B`。
- **L1501 EN**: Declares TableGen def record `V6_vmpyowh_rnd_sacc`.
  **L1501 CN**: 声明 TableGen def 记录 `V6_vmpyowh_rnd_sacc`。
- **L1502 EN**: Declares TableGen def record `V6_vmpyowh_rnd_sacc_128B`.
  **L1502 CN**: 声明 TableGen def 记录 `V6_vmpyowh_rnd_sacc_128B`。
- **L1503 EN**: Declares TableGen def record `V6_vmpyowh_sacc`.
  **L1503 CN**: 声明 TableGen def 记录 `V6_vmpyowh_sacc`。
- **L1504 EN**: Declares TableGen def record `V6_vmpyowh_sacc_128B`.
  **L1504 CN**: 声明 TableGen def 记录 `V6_vmpyowh_sacc_128B`。
- **L1505 EN**: Declares TableGen def record `V6_vmpyub`.
  **L1505 CN**: 声明 TableGen def 记录 `V6_vmpyub`。
- **L1506 EN**: Declares TableGen def record `V6_vmpyub_128B`.
  **L1506 CN**: 声明 TableGen def 记录 `V6_vmpyub_128B`。
- **L1507 EN**: Declares TableGen def record `V6_vmpyub_acc`.
  **L1507 CN**: 声明 TableGen def 记录 `V6_vmpyub_acc`。
- **L1508 EN**: Declares TableGen def record `V6_vmpyub_acc_128B`.
  **L1508 CN**: 声明 TableGen def 记录 `V6_vmpyub_acc_128B`。
- **L1509 EN**: Declares TableGen def record `V6_vmpyubv`.
  **L1509 CN**: 声明 TableGen def 记录 `V6_vmpyubv`。
- **L1510 EN**: Declares TableGen def record `V6_vmpyubv_128B`.
  **L1510 CN**: 声明 TableGen def 记录 `V6_vmpyubv_128B`。
- **L1511 EN**: Declares TableGen def record `V6_vmpyubv_acc`.
  **L1511 CN**: 声明 TableGen def 记录 `V6_vmpyubv_acc`。
- **L1512 EN**: Declares TableGen def record `V6_vmpyubv_acc_128B`.
  **L1512 CN**: 声明 TableGen def 记录 `V6_vmpyubv_acc_128B`。

### Lines 1513-1540

````tablegen
  def V6_vmpyuh : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>, int)">;
  def V6_vmpyuh_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>, int)">;
  def V6_vmpyuh_acc : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<16, int>, int)">;
  def V6_vmpyuh_acc_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<32, int>, int)">;
  def V6_vmpyuhv : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpyuhv_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpyuhv_acc : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpyuhv_acc_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vmux : HexagonBuiltin<"_Vector<16, int>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vmux_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vnavgh : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vnavgh_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vnavgub : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vnavgub_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vnavgw : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vnavgw_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vnormamth : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vnormamth_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vnormamtw : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vnormamtw_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vnot : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vnot_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vor : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vor_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vpackeb : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vpackeb_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vpackeh : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vpackeh_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
````
- **L1513 EN**: Declares TableGen def record `V6_vmpyuh`.
  **L1513 CN**: 声明 TableGen def 记录 `V6_vmpyuh`。
- **L1514 EN**: Declares TableGen def record `V6_vmpyuh_128B`.
  **L1514 CN**: 声明 TableGen def 记录 `V6_vmpyuh_128B`。
- **L1515 EN**: Declares TableGen def record `V6_vmpyuh_acc`.
  **L1515 CN**: 声明 TableGen def 记录 `V6_vmpyuh_acc`。
- **L1516 EN**: Declares TableGen def record `V6_vmpyuh_acc_128B`.
  **L1516 CN**: 声明 TableGen def 记录 `V6_vmpyuh_acc_128B`。
- **L1517 EN**: Declares TableGen def record `V6_vmpyuhv`.
  **L1517 CN**: 声明 TableGen def 记录 `V6_vmpyuhv`。
- **L1518 EN**: Declares TableGen def record `V6_vmpyuhv_128B`.
  **L1518 CN**: 声明 TableGen def 记录 `V6_vmpyuhv_128B`。
- **L1519 EN**: Declares TableGen def record `V6_vmpyuhv_acc`.
  **L1519 CN**: 声明 TableGen def 记录 `V6_vmpyuhv_acc`。
- **L1520 EN**: Declares TableGen def record `V6_vmpyuhv_acc_128B`.
  **L1520 CN**: 声明 TableGen def 记录 `V6_vmpyuhv_acc_128B`。
- **L1521 EN**: Declares TableGen def record `V6_vmux`.
  **L1521 CN**: 声明 TableGen def 记录 `V6_vmux`。
- **L1522 EN**: Declares TableGen def record `V6_vmux_128B`.
  **L1522 CN**: 声明 TableGen def 记录 `V6_vmux_128B`。
- **L1523 EN**: Declares TableGen def record `V6_vnavgh`.
  **L1523 CN**: 声明 TableGen def 记录 `V6_vnavgh`。
- **L1524 EN**: Declares TableGen def record `V6_vnavgh_128B`.
  **L1524 CN**: 声明 TableGen def 记录 `V6_vnavgh_128B`。
- **L1525 EN**: Declares TableGen def record `V6_vnavgub`.
  **L1525 CN**: 声明 TableGen def 记录 `V6_vnavgub`。
- **L1526 EN**: Declares TableGen def record `V6_vnavgub_128B`.
  **L1526 CN**: 声明 TableGen def 记录 `V6_vnavgub_128B`。
- **L1527 EN**: Declares TableGen def record `V6_vnavgw`.
  **L1527 CN**: 声明 TableGen def 记录 `V6_vnavgw`。
- **L1528 EN**: Declares TableGen def record `V6_vnavgw_128B`.
  **L1528 CN**: 声明 TableGen def 记录 `V6_vnavgw_128B`。
- **L1529 EN**: Declares TableGen def record `V6_vnormamth`.
  **L1529 CN**: 声明 TableGen def 记录 `V6_vnormamth`。
- **L1530 EN**: Declares TableGen def record `V6_vnormamth_128B`.
  **L1530 CN**: 声明 TableGen def 记录 `V6_vnormamth_128B`。
- **L1531 EN**: Declares TableGen def record `V6_vnormamtw`.
  **L1531 CN**: 声明 TableGen def 记录 `V6_vnormamtw`。
- **L1532 EN**: Declares TableGen def record `V6_vnormamtw_128B`.
  **L1532 CN**: 声明 TableGen def 记录 `V6_vnormamtw_128B`。
- **L1533 EN**: Declares TableGen def record `V6_vnot`.
  **L1533 CN**: 声明 TableGen def 记录 `V6_vnot`。
- **L1534 EN**: Declares TableGen def record `V6_vnot_128B`.
  **L1534 CN**: 声明 TableGen def 记录 `V6_vnot_128B`。
- **L1535 EN**: Declares TableGen def record `V6_vor`.
  **L1535 CN**: 声明 TableGen def 记录 `V6_vor`。
- **L1536 EN**: Declares TableGen def record `V6_vor_128B`.
  **L1536 CN**: 声明 TableGen def 记录 `V6_vor_128B`。
- **L1537 EN**: Declares TableGen def record `V6_vpackeb`.
  **L1537 CN**: 声明 TableGen def 记录 `V6_vpackeb`。
- **L1538 EN**: Declares TableGen def record `V6_vpackeb_128B`.
  **L1538 CN**: 声明 TableGen def 记录 `V6_vpackeb_128B`。
- **L1539 EN**: Declares TableGen def record `V6_vpackeh`.
  **L1539 CN**: 声明 TableGen def 记录 `V6_vpackeh`。
- **L1540 EN**: Declares TableGen def record `V6_vpackeh_128B`.
  **L1540 CN**: 声明 TableGen def 记录 `V6_vpackeh_128B`。

### Lines 1541-1568

````tablegen
  def V6_vpackhb_sat : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vpackhb_sat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vpackhub_sat : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vpackhub_sat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vpackob : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vpackob_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vpackoh : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vpackoh_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vpackwh_sat : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vpackwh_sat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vpackwuh_sat : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vpackwuh_sat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vpopcounth : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vpopcounth_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vrdelta : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vrdelta_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vrmpybus : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, int)">;
  def V6_vrmpybus_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, int)">;
  def V6_vrmpybus_acc : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, int)">;
  def V6_vrmpybus_acc_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vrmpybusi : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, int, unsigned _Constant int)">;
  def V6_vrmpybusi_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, int, unsigned _Constant int)">;
  def V6_vrmpybusi_acc : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int, unsigned _Constant int)">;
  def V6_vrmpybusi_acc_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<64, int>, int, unsigned _Constant int)">;
  def V6_vrmpybusv : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vrmpybusv_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vrmpybusv_acc : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vrmpybusv_acc_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, _Vector<32, int>)">;
````
- **L1541 EN**: Declares TableGen def record `V6_vpackhb_sat`.
  **L1541 CN**: 声明 TableGen def 记录 `V6_vpackhb_sat`。
- **L1542 EN**: Declares TableGen def record `V6_vpackhb_sat_128B`.
  **L1542 CN**: 声明 TableGen def 记录 `V6_vpackhb_sat_128B`。
- **L1543 EN**: Declares TableGen def record `V6_vpackhub_sat`.
  **L1543 CN**: 声明 TableGen def 记录 `V6_vpackhub_sat`。
- **L1544 EN**: Declares TableGen def record `V6_vpackhub_sat_128B`.
  **L1544 CN**: 声明 TableGen def 记录 `V6_vpackhub_sat_128B`。
- **L1545 EN**: Declares TableGen def record `V6_vpackob`.
  **L1545 CN**: 声明 TableGen def 记录 `V6_vpackob`。
- **L1546 EN**: Declares TableGen def record `V6_vpackob_128B`.
  **L1546 CN**: 声明 TableGen def 记录 `V6_vpackob_128B`。
- **L1547 EN**: Declares TableGen def record `V6_vpackoh`.
  **L1547 CN**: 声明 TableGen def 记录 `V6_vpackoh`。
- **L1548 EN**: Declares TableGen def record `V6_vpackoh_128B`.
  **L1548 CN**: 声明 TableGen def 记录 `V6_vpackoh_128B`。
- **L1549 EN**: Declares TableGen def record `V6_vpackwh_sat`.
  **L1549 CN**: 声明 TableGen def 记录 `V6_vpackwh_sat`。
- **L1550 EN**: Declares TableGen def record `V6_vpackwh_sat_128B`.
  **L1550 CN**: 声明 TableGen def 记录 `V6_vpackwh_sat_128B`。
- **L1551 EN**: Declares TableGen def record `V6_vpackwuh_sat`.
  **L1551 CN**: 声明 TableGen def 记录 `V6_vpackwuh_sat`。
- **L1552 EN**: Declares TableGen def record `V6_vpackwuh_sat_128B`.
  **L1552 CN**: 声明 TableGen def 记录 `V6_vpackwuh_sat_128B`。
- **L1553 EN**: Declares TableGen def record `V6_vpopcounth`.
  **L1553 CN**: 声明 TableGen def 记录 `V6_vpopcounth`。
- **L1554 EN**: Declares TableGen def record `V6_vpopcounth_128B`.
  **L1554 CN**: 声明 TableGen def 记录 `V6_vpopcounth_128B`。
- **L1555 EN**: Declares TableGen def record `V6_vrdelta`.
  **L1555 CN**: 声明 TableGen def 记录 `V6_vrdelta`。
- **L1556 EN**: Declares TableGen def record `V6_vrdelta_128B`.
  **L1556 CN**: 声明 TableGen def 记录 `V6_vrdelta_128B`。
- **L1557 EN**: Declares TableGen def record `V6_vrmpybus`.
  **L1557 CN**: 声明 TableGen def 记录 `V6_vrmpybus`。
- **L1558 EN**: Declares TableGen def record `V6_vrmpybus_128B`.
  **L1558 CN**: 声明 TableGen def 记录 `V6_vrmpybus_128B`。
- **L1559 EN**: Declares TableGen def record `V6_vrmpybus_acc`.
  **L1559 CN**: 声明 TableGen def 记录 `V6_vrmpybus_acc`。
- **L1560 EN**: Declares TableGen def record `V6_vrmpybus_acc_128B`.
  **L1560 CN**: 声明 TableGen def 记录 `V6_vrmpybus_acc_128B`。
- **L1561 EN**: Declares TableGen def record `V6_vrmpybusi`.
  **L1561 CN**: 声明 TableGen def 记录 `V6_vrmpybusi`。
- **L1562 EN**: Declares TableGen def record `V6_vrmpybusi_128B`.
  **L1562 CN**: 声明 TableGen def 记录 `V6_vrmpybusi_128B`。
- **L1563 EN**: Declares TableGen def record `V6_vrmpybusi_acc`.
  **L1563 CN**: 声明 TableGen def 记录 `V6_vrmpybusi_acc`。
- **L1564 EN**: Declares TableGen def record `V6_vrmpybusi_acc_128B`.
  **L1564 CN**: 声明 TableGen def 记录 `V6_vrmpybusi_acc_128B`。
- **L1565 EN**: Declares TableGen def record `V6_vrmpybusv`.
  **L1565 CN**: 声明 TableGen def 记录 `V6_vrmpybusv`。
- **L1566 EN**: Declares TableGen def record `V6_vrmpybusv_128B`.
  **L1566 CN**: 声明 TableGen def 记录 `V6_vrmpybusv_128B`。
- **L1567 EN**: Declares TableGen def record `V6_vrmpybusv_acc`.
  **L1567 CN**: 声明 TableGen def 记录 `V6_vrmpybusv_acc`。
- **L1568 EN**: Declares TableGen def record `V6_vrmpybusv_acc_128B`.
  **L1568 CN**: 声明 TableGen def 记录 `V6_vrmpybusv_acc_128B`。

### Lines 1569-1596

````tablegen
  def V6_vrmpybv : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vrmpybv_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vrmpybv_acc : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vrmpybv_acc_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vrmpyub : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, int)">;
  def V6_vrmpyub_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, int)">;
  def V6_vrmpyub_acc : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, int)">;
  def V6_vrmpyub_acc_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vrmpyubi : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, int, unsigned _Constant int)">;
  def V6_vrmpyubi_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, int, unsigned _Constant int)">;
  def V6_vrmpyubi_acc : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int, unsigned _Constant int)">;
  def V6_vrmpyubi_acc_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<64, int>, int, unsigned _Constant int)">;
  def V6_vrmpyubv : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vrmpyubv_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vrmpyubv_acc : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vrmpyubv_acc_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vror : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, int)">;
  def V6_vror_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, int)">;
  def V6_vroundhb : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vroundhb_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vroundhub : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vroundhub_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vroundwh : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vroundwh_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vroundwuh : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vroundwuh_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vrsadubi : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, int, unsigned _Constant int)">;
  def V6_vrsadubi_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, int, unsigned _Constant int)">;
````
- **L1569 EN**: Declares TableGen def record `V6_vrmpybv`.
  **L1569 CN**: 声明 TableGen def 记录 `V6_vrmpybv`。
- **L1570 EN**: Declares TableGen def record `V6_vrmpybv_128B`.
  **L1570 CN**: 声明 TableGen def 记录 `V6_vrmpybv_128B`。
- **L1571 EN**: Declares TableGen def record `V6_vrmpybv_acc`.
  **L1571 CN**: 声明 TableGen def 记录 `V6_vrmpybv_acc`。
- **L1572 EN**: Declares TableGen def record `V6_vrmpybv_acc_128B`.
  **L1572 CN**: 声明 TableGen def 记录 `V6_vrmpybv_acc_128B`。
- **L1573 EN**: Declares TableGen def record `V6_vrmpyub`.
  **L1573 CN**: 声明 TableGen def 记录 `V6_vrmpyub`。
- **L1574 EN**: Declares TableGen def record `V6_vrmpyub_128B`.
  **L1574 CN**: 声明 TableGen def 记录 `V6_vrmpyub_128B`。
- **L1575 EN**: Declares TableGen def record `V6_vrmpyub_acc`.
  **L1575 CN**: 声明 TableGen def 记录 `V6_vrmpyub_acc`。
- **L1576 EN**: Declares TableGen def record `V6_vrmpyub_acc_128B`.
  **L1576 CN**: 声明 TableGen def 记录 `V6_vrmpyub_acc_128B`。
- **L1577 EN**: Declares TableGen def record `V6_vrmpyubi`.
  **L1577 CN**: 声明 TableGen def 记录 `V6_vrmpyubi`。
- **L1578 EN**: Declares TableGen def record `V6_vrmpyubi_128B`.
  **L1578 CN**: 声明 TableGen def 记录 `V6_vrmpyubi_128B`。
- **L1579 EN**: Declares TableGen def record `V6_vrmpyubi_acc`.
  **L1579 CN**: 声明 TableGen def 记录 `V6_vrmpyubi_acc`。
- **L1580 EN**: Declares TableGen def record `V6_vrmpyubi_acc_128B`.
  **L1580 CN**: 声明 TableGen def 记录 `V6_vrmpyubi_acc_128B`。
- **L1581 EN**: Declares TableGen def record `V6_vrmpyubv`.
  **L1581 CN**: 声明 TableGen def 记录 `V6_vrmpyubv`。
- **L1582 EN**: Declares TableGen def record `V6_vrmpyubv_128B`.
  **L1582 CN**: 声明 TableGen def 记录 `V6_vrmpyubv_128B`。
- **L1583 EN**: Declares TableGen def record `V6_vrmpyubv_acc`.
  **L1583 CN**: 声明 TableGen def 记录 `V6_vrmpyubv_acc`。
- **L1584 EN**: Declares TableGen def record `V6_vrmpyubv_acc_128B`.
  **L1584 CN**: 声明 TableGen def 记录 `V6_vrmpyubv_acc_128B`。
- **L1585 EN**: Declares TableGen def record `V6_vror`.
  **L1585 CN**: 声明 TableGen def 记录 `V6_vror`。
- **L1586 EN**: Declares TableGen def record `V6_vror_128B`.
  **L1586 CN**: 声明 TableGen def 记录 `V6_vror_128B`。
- **L1587 EN**: Declares TableGen def record `V6_vroundhb`.
  **L1587 CN**: 声明 TableGen def 记录 `V6_vroundhb`。
- **L1588 EN**: Declares TableGen def record `V6_vroundhb_128B`.
  **L1588 CN**: 声明 TableGen def 记录 `V6_vroundhb_128B`。
- **L1589 EN**: Declares TableGen def record `V6_vroundhub`.
  **L1589 CN**: 声明 TableGen def 记录 `V6_vroundhub`。
- **L1590 EN**: Declares TableGen def record `V6_vroundhub_128B`.
  **L1590 CN**: 声明 TableGen def 记录 `V6_vroundhub_128B`。
- **L1591 EN**: Declares TableGen def record `V6_vroundwh`.
  **L1591 CN**: 声明 TableGen def 记录 `V6_vroundwh`。
- **L1592 EN**: Declares TableGen def record `V6_vroundwh_128B`.
  **L1592 CN**: 声明 TableGen def 记录 `V6_vroundwh_128B`。
- **L1593 EN**: Declares TableGen def record `V6_vroundwuh`.
  **L1593 CN**: 声明 TableGen def 记录 `V6_vroundwuh`。
- **L1594 EN**: Declares TableGen def record `V6_vroundwuh_128B`.
  **L1594 CN**: 声明 TableGen def 记录 `V6_vroundwuh_128B`。
- **L1595 EN**: Declares TableGen def record `V6_vrsadubi`.
  **L1595 CN**: 声明 TableGen def 记录 `V6_vrsadubi`。
- **L1596 EN**: Declares TableGen def record `V6_vrsadubi_128B`.
  **L1596 CN**: 声明 TableGen def 记录 `V6_vrsadubi_128B`。

### Lines 1597-1624

````tablegen
  def V6_vrsadubi_acc : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int, unsigned _Constant int)">;
  def V6_vrsadubi_acc_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<64, int>, int, unsigned _Constant int)">;
  def V6_vsathub : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vsathub_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vsatwh : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vsatwh_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vsb : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>)">;
  def V6_vsb_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>)">;
  def V6_vsh : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>)">;
  def V6_vsh_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>)">;
  def V6_vshufeh : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vshufeh_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vshuffb : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vshuffb_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vshuffeb : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vshuffeb_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vshuffh : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vshuffh_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vshuffob : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vshuffob_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vshuffvdd : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>, _Vector<16, int>, int)">;
  def V6_vshuffvdd_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vshufoeb : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vshufoeb_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vshufoeh : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vshufoeh_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vshufoh : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vshufoh_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
````
- **L1597 EN**: Declares TableGen def record `V6_vrsadubi_acc`.
  **L1597 CN**: 声明 TableGen def 记录 `V6_vrsadubi_acc`。
- **L1598 EN**: Declares TableGen def record `V6_vrsadubi_acc_128B`.
  **L1598 CN**: 声明 TableGen def 记录 `V6_vrsadubi_acc_128B`。
- **L1599 EN**: Declares TableGen def record `V6_vsathub`.
  **L1599 CN**: 声明 TableGen def 记录 `V6_vsathub`。
- **L1600 EN**: Declares TableGen def record `V6_vsathub_128B`.
  **L1600 CN**: 声明 TableGen def 记录 `V6_vsathub_128B`。
- **L1601 EN**: Declares TableGen def record `V6_vsatwh`.
  **L1601 CN**: 声明 TableGen def 记录 `V6_vsatwh`。
- **L1602 EN**: Declares TableGen def record `V6_vsatwh_128B`.
  **L1602 CN**: 声明 TableGen def 记录 `V6_vsatwh_128B`。
- **L1603 EN**: Declares TableGen def record `V6_vsb`.
  **L1603 CN**: 声明 TableGen def 记录 `V6_vsb`。
- **L1604 EN**: Declares TableGen def record `V6_vsb_128B`.
  **L1604 CN**: 声明 TableGen def 记录 `V6_vsb_128B`。
- **L1605 EN**: Declares TableGen def record `V6_vsh`.
  **L1605 CN**: 声明 TableGen def 记录 `V6_vsh`。
- **L1606 EN**: Declares TableGen def record `V6_vsh_128B`.
  **L1606 CN**: 声明 TableGen def 记录 `V6_vsh_128B`。
- **L1607 EN**: Declares TableGen def record `V6_vshufeh`.
  **L1607 CN**: 声明 TableGen def 记录 `V6_vshufeh`。
- **L1608 EN**: Declares TableGen def record `V6_vshufeh_128B`.
  **L1608 CN**: 声明 TableGen def 记录 `V6_vshufeh_128B`。
- **L1609 EN**: Declares TableGen def record `V6_vshuffb`.
  **L1609 CN**: 声明 TableGen def 记录 `V6_vshuffb`。
- **L1610 EN**: Declares TableGen def record `V6_vshuffb_128B`.
  **L1610 CN**: 声明 TableGen def 记录 `V6_vshuffb_128B`。
- **L1611 EN**: Declares TableGen def record `V6_vshuffeb`.
  **L1611 CN**: 声明 TableGen def 记录 `V6_vshuffeb`。
- **L1612 EN**: Declares TableGen def record `V6_vshuffeb_128B`.
  **L1612 CN**: 声明 TableGen def 记录 `V6_vshuffeb_128B`。
- **L1613 EN**: Declares TableGen def record `V6_vshuffh`.
  **L1613 CN**: 声明 TableGen def 记录 `V6_vshuffh`。
- **L1614 EN**: Declares TableGen def record `V6_vshuffh_128B`.
  **L1614 CN**: 声明 TableGen def 记录 `V6_vshuffh_128B`。
- **L1615 EN**: Declares TableGen def record `V6_vshuffob`.
  **L1615 CN**: 声明 TableGen def 记录 `V6_vshuffob`。
- **L1616 EN**: Declares TableGen def record `V6_vshuffob_128B`.
  **L1616 CN**: 声明 TableGen def 记录 `V6_vshuffob_128B`。
- **L1617 EN**: Declares TableGen def record `V6_vshuffvdd`.
  **L1617 CN**: 声明 TableGen def 记录 `V6_vshuffvdd`。
- **L1618 EN**: Declares TableGen def record `V6_vshuffvdd_128B`.
  **L1618 CN**: 声明 TableGen def 记录 `V6_vshuffvdd_128B`。
- **L1619 EN**: Declares TableGen def record `V6_vshufoeb`.
  **L1619 CN**: 声明 TableGen def 记录 `V6_vshufoeb`。
- **L1620 EN**: Declares TableGen def record `V6_vshufoeb_128B`.
  **L1620 CN**: 声明 TableGen def 记录 `V6_vshufoeb_128B`。
- **L1621 EN**: Declares TableGen def record `V6_vshufoeh`.
  **L1621 CN**: 声明 TableGen def 记录 `V6_vshufoeh`。
- **L1622 EN**: Declares TableGen def record `V6_vshufoeh_128B`.
  **L1622 CN**: 声明 TableGen def 记录 `V6_vshufoeh_128B`。
- **L1623 EN**: Declares TableGen def record `V6_vshufoh`.
  **L1623 CN**: 声明 TableGen def 记录 `V6_vshufoh`。
- **L1624 EN**: Declares TableGen def record `V6_vshufoh_128B`.
  **L1624 CN**: 声明 TableGen def 记录 `V6_vshufoh_128B`。

### Lines 1625-1652

````tablegen
  def V6_vsubb : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vsubb_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vsubb_dv : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vsubb_dv_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<64, int>)">;
  def V6_vsubbnq : HexagonBuiltin<"_Vector<16, int>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vsubbnq_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vsubbq : HexagonBuiltin<"_Vector<16, int>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vsubbq_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vsubh : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vsubh_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vsubh_dv : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vsubh_dv_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<64, int>)">;
  def V6_vsubhnq : HexagonBuiltin<"_Vector<16, int>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vsubhnq_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vsubhq : HexagonBuiltin<"_Vector<16, int>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vsubhq_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vsubhsat : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vsubhsat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vsubhsat_dv : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vsubhsat_dv_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<64, int>)">;
  def V6_vsubhw : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vsubhw_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vsububh : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vsububh_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vsububsat : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vsububsat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vsububsat_dv : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vsububsat_dv_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<64, int>)">;
````
- **L1625 EN**: Declares TableGen def record `V6_vsubb`.
  **L1625 CN**: 声明 TableGen def 记录 `V6_vsubb`。
- **L1626 EN**: Declares TableGen def record `V6_vsubb_128B`.
  **L1626 CN**: 声明 TableGen def 记录 `V6_vsubb_128B`。
- **L1627 EN**: Declares TableGen def record `V6_vsubb_dv`.
  **L1627 CN**: 声明 TableGen def 记录 `V6_vsubb_dv`。
- **L1628 EN**: Declares TableGen def record `V6_vsubb_dv_128B`.
  **L1628 CN**: 声明 TableGen def 记录 `V6_vsubb_dv_128B`。
- **L1629 EN**: Declares TableGen def record `V6_vsubbnq`.
  **L1629 CN**: 声明 TableGen def 记录 `V6_vsubbnq`。
- **L1630 EN**: Declares TableGen def record `V6_vsubbnq_128B`.
  **L1630 CN**: 声明 TableGen def 记录 `V6_vsubbnq_128B`。
- **L1631 EN**: Declares TableGen def record `V6_vsubbq`.
  **L1631 CN**: 声明 TableGen def 记录 `V6_vsubbq`。
- **L1632 EN**: Declares TableGen def record `V6_vsubbq_128B`.
  **L1632 CN**: 声明 TableGen def 记录 `V6_vsubbq_128B`。
- **L1633 EN**: Declares TableGen def record `V6_vsubh`.
  **L1633 CN**: 声明 TableGen def 记录 `V6_vsubh`。
- **L1634 EN**: Declares TableGen def record `V6_vsubh_128B`.
  **L1634 CN**: 声明 TableGen def 记录 `V6_vsubh_128B`。
- **L1635 EN**: Declares TableGen def record `V6_vsubh_dv`.
  **L1635 CN**: 声明 TableGen def 记录 `V6_vsubh_dv`。
- **L1636 EN**: Declares TableGen def record `V6_vsubh_dv_128B`.
  **L1636 CN**: 声明 TableGen def 记录 `V6_vsubh_dv_128B`。
- **L1637 EN**: Declares TableGen def record `V6_vsubhnq`.
  **L1637 CN**: 声明 TableGen def 记录 `V6_vsubhnq`。
- **L1638 EN**: Declares TableGen def record `V6_vsubhnq_128B`.
  **L1638 CN**: 声明 TableGen def 记录 `V6_vsubhnq_128B`。
- **L1639 EN**: Declares TableGen def record `V6_vsubhq`.
  **L1639 CN**: 声明 TableGen def 记录 `V6_vsubhq`。
- **L1640 EN**: Declares TableGen def record `V6_vsubhq_128B`.
  **L1640 CN**: 声明 TableGen def 记录 `V6_vsubhq_128B`。
- **L1641 EN**: Declares TableGen def record `V6_vsubhsat`.
  **L1641 CN**: 声明 TableGen def 记录 `V6_vsubhsat`。
- **L1642 EN**: Declares TableGen def record `V6_vsubhsat_128B`.
  **L1642 CN**: 声明 TableGen def 记录 `V6_vsubhsat_128B`。
- **L1643 EN**: Declares TableGen def record `V6_vsubhsat_dv`.
  **L1643 CN**: 声明 TableGen def 记录 `V6_vsubhsat_dv`。
- **L1644 EN**: Declares TableGen def record `V6_vsubhsat_dv_128B`.
  **L1644 CN**: 声明 TableGen def 记录 `V6_vsubhsat_dv_128B`。
- **L1645 EN**: Declares TableGen def record `V6_vsubhw`.
  **L1645 CN**: 声明 TableGen def 记录 `V6_vsubhw`。
- **L1646 EN**: Declares TableGen def record `V6_vsubhw_128B`.
  **L1646 CN**: 声明 TableGen def 记录 `V6_vsubhw_128B`。
- **L1647 EN**: Declares TableGen def record `V6_vsububh`.
  **L1647 CN**: 声明 TableGen def 记录 `V6_vsububh`。
- **L1648 EN**: Declares TableGen def record `V6_vsububh_128B`.
  **L1648 CN**: 声明 TableGen def 记录 `V6_vsububh_128B`。
- **L1649 EN**: Declares TableGen def record `V6_vsububsat`.
  **L1649 CN**: 声明 TableGen def 记录 `V6_vsububsat`。
- **L1650 EN**: Declares TableGen def record `V6_vsububsat_128B`.
  **L1650 CN**: 声明 TableGen def 记录 `V6_vsububsat_128B`。
- **L1651 EN**: Declares TableGen def record `V6_vsububsat_dv`.
  **L1651 CN**: 声明 TableGen def 记录 `V6_vsububsat_dv`。
- **L1652 EN**: Declares TableGen def record `V6_vsububsat_dv_128B`.
  **L1652 CN**: 声明 TableGen def 记录 `V6_vsububsat_dv_128B`。

### Lines 1653-1680

````tablegen
  def V6_vsubuhsat : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vsubuhsat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vsubuhsat_dv : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vsubuhsat_dv_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<64, int>)">;
  def V6_vsubuhw : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vsubuhw_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vsubw : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vsubw_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vsubw_dv : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vsubw_dv_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<64, int>)">;
  def V6_vsubwnq : HexagonBuiltin<"_Vector<16, int>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vsubwnq_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vsubwq : HexagonBuiltin<"_Vector<16, int>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vsubwq_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vsubwsat : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vsubwsat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vsubwsat_dv : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vsubwsat_dv_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<64, int>)">;
  def V6_vswap : HexagonBuiltin<"_Vector<32, int>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vswap_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vtmpyb : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, int)">;
  def V6_vtmpyb_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, int)">;
  def V6_vtmpyb_acc : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vtmpyb_acc_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<64, int>, int)">;
  def V6_vtmpybus : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, int)">;
  def V6_vtmpybus_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, int)">;
  def V6_vtmpybus_acc : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vtmpybus_acc_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<64, int>, int)">;
````
- **L1653 EN**: Declares TableGen def record `V6_vsubuhsat`.
  **L1653 CN**: 声明 TableGen def 记录 `V6_vsubuhsat`。
- **L1654 EN**: Declares TableGen def record `V6_vsubuhsat_128B`.
  **L1654 CN**: 声明 TableGen def 记录 `V6_vsubuhsat_128B`。
- **L1655 EN**: Declares TableGen def record `V6_vsubuhsat_dv`.
  **L1655 CN**: 声明 TableGen def 记录 `V6_vsubuhsat_dv`。
- **L1656 EN**: Declares TableGen def record `V6_vsubuhsat_dv_128B`.
  **L1656 CN**: 声明 TableGen def 记录 `V6_vsubuhsat_dv_128B`。
- **L1657 EN**: Declares TableGen def record `V6_vsubuhw`.
  **L1657 CN**: 声明 TableGen def 记录 `V6_vsubuhw`。
- **L1658 EN**: Declares TableGen def record `V6_vsubuhw_128B`.
  **L1658 CN**: 声明 TableGen def 记录 `V6_vsubuhw_128B`。
- **L1659 EN**: Declares TableGen def record `V6_vsubw`.
  **L1659 CN**: 声明 TableGen def 记录 `V6_vsubw`。
- **L1660 EN**: Declares TableGen def record `V6_vsubw_128B`.
  **L1660 CN**: 声明 TableGen def 记录 `V6_vsubw_128B`。
- **L1661 EN**: Declares TableGen def record `V6_vsubw_dv`.
  **L1661 CN**: 声明 TableGen def 记录 `V6_vsubw_dv`。
- **L1662 EN**: Declares TableGen def record `V6_vsubw_dv_128B`.
  **L1662 CN**: 声明 TableGen def 记录 `V6_vsubw_dv_128B`。
- **L1663 EN**: Declares TableGen def record `V6_vsubwnq`.
  **L1663 CN**: 声明 TableGen def 记录 `V6_vsubwnq`。
- **L1664 EN**: Declares TableGen def record `V6_vsubwnq_128B`.
  **L1664 CN**: 声明 TableGen def 记录 `V6_vsubwnq_128B`。
- **L1665 EN**: Declares TableGen def record `V6_vsubwq`.
  **L1665 CN**: 声明 TableGen def 记录 `V6_vsubwq`。
- **L1666 EN**: Declares TableGen def record `V6_vsubwq_128B`.
  **L1666 CN**: 声明 TableGen def 记录 `V6_vsubwq_128B`。
- **L1667 EN**: Declares TableGen def record `V6_vsubwsat`.
  **L1667 CN**: 声明 TableGen def 记录 `V6_vsubwsat`。
- **L1668 EN**: Declares TableGen def record `V6_vsubwsat_128B`.
  **L1668 CN**: 声明 TableGen def 记录 `V6_vsubwsat_128B`。
- **L1669 EN**: Declares TableGen def record `V6_vsubwsat_dv`.
  **L1669 CN**: 声明 TableGen def 记录 `V6_vsubwsat_dv`。
- **L1670 EN**: Declares TableGen def record `V6_vsubwsat_dv_128B`.
  **L1670 CN**: 声明 TableGen def 记录 `V6_vsubwsat_dv_128B`。
- **L1671 EN**: Declares TableGen def record `V6_vswap`.
  **L1671 CN**: 声明 TableGen def 记录 `V6_vswap`。
- **L1672 EN**: Declares TableGen def record `V6_vswap_128B`.
  **L1672 CN**: 声明 TableGen def 记录 `V6_vswap_128B`。
- **L1673 EN**: Declares TableGen def record `V6_vtmpyb`.
  **L1673 CN**: 声明 TableGen def 记录 `V6_vtmpyb`。
- **L1674 EN**: Declares TableGen def record `V6_vtmpyb_128B`.
  **L1674 CN**: 声明 TableGen def 记录 `V6_vtmpyb_128B`。
- **L1675 EN**: Declares TableGen def record `V6_vtmpyb_acc`.
  **L1675 CN**: 声明 TableGen def 记录 `V6_vtmpyb_acc`。
- **L1676 EN**: Declares TableGen def record `V6_vtmpyb_acc_128B`.
  **L1676 CN**: 声明 TableGen def 记录 `V6_vtmpyb_acc_128B`。
- **L1677 EN**: Declares TableGen def record `V6_vtmpybus`.
  **L1677 CN**: 声明 TableGen def 记录 `V6_vtmpybus`。
- **L1678 EN**: Declares TableGen def record `V6_vtmpybus_128B`.
  **L1678 CN**: 声明 TableGen def 记录 `V6_vtmpybus_128B`。
- **L1679 EN**: Declares TableGen def record `V6_vtmpybus_acc`.
  **L1679 CN**: 声明 TableGen def 记录 `V6_vtmpybus_acc`。
- **L1680 EN**: Declares TableGen def record `V6_vtmpybus_acc_128B`.
  **L1680 CN**: 声明 TableGen def 记录 `V6_vtmpybus_acc_128B`。

### Lines 1681-1708

````tablegen
  def V6_vtmpyhb : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, int)">;
  def V6_vtmpyhb_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, int)">;
  def V6_vtmpyhb_acc : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vtmpyhb_acc_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<64, int>, int)">;
  def V6_vunpackb : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>)">;
  def V6_vunpackb_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>)">;
  def V6_vunpackh : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>)">;
  def V6_vunpackh_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>)">;
  def V6_vunpackob : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<16, int>)">;
  def V6_vunpackob_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<32, int>)">;
  def V6_vunpackoh : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<16, int>)">;
  def V6_vunpackoh_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<32, int>)">;
  def V6_vunpackub : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>)">;
  def V6_vunpackub_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>)">;
  def V6_vunpackuh : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>)">;
  def V6_vunpackuh_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>)">;
  def V6_vxor : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vxor_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vzb : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>)">;
  def V6_vzb_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>)">;
  def V6_vzh : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>)">;
  def V6_vzh_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>)">;
}

// V62 HVX Instructions.

let Features = HVXV62.Features in {
  def V6_lvsplatb : HexagonBuiltin<"_Vector<16, int>(int)">;
````
- **L1681 EN**: Declares TableGen def record `V6_vtmpyhb`.
  **L1681 CN**: 声明 TableGen def 记录 `V6_vtmpyhb`。
- **L1682 EN**: Declares TableGen def record `V6_vtmpyhb_128B`.
  **L1682 CN**: 声明 TableGen def 记录 `V6_vtmpyhb_128B`。
- **L1683 EN**: Declares TableGen def record `V6_vtmpyhb_acc`.
  **L1683 CN**: 声明 TableGen def 记录 `V6_vtmpyhb_acc`。
- **L1684 EN**: Declares TableGen def record `V6_vtmpyhb_acc_128B`.
  **L1684 CN**: 声明 TableGen def 记录 `V6_vtmpyhb_acc_128B`。
- **L1685 EN**: Declares TableGen def record `V6_vunpackb`.
  **L1685 CN**: 声明 TableGen def 记录 `V6_vunpackb`。
- **L1686 EN**: Declares TableGen def record `V6_vunpackb_128B`.
  **L1686 CN**: 声明 TableGen def 记录 `V6_vunpackb_128B`。
- **L1687 EN**: Declares TableGen def record `V6_vunpackh`.
  **L1687 CN**: 声明 TableGen def 记录 `V6_vunpackh`。
- **L1688 EN**: Declares TableGen def record `V6_vunpackh_128B`.
  **L1688 CN**: 声明 TableGen def 记录 `V6_vunpackh_128B`。
- **L1689 EN**: Declares TableGen def record `V6_vunpackob`.
  **L1689 CN**: 声明 TableGen def 记录 `V6_vunpackob`。
- **L1690 EN**: Declares TableGen def record `V6_vunpackob_128B`.
  **L1690 CN**: 声明 TableGen def 记录 `V6_vunpackob_128B`。
- **L1691 EN**: Declares TableGen def record `V6_vunpackoh`.
  **L1691 CN**: 声明 TableGen def 记录 `V6_vunpackoh`。
- **L1692 EN**: Declares TableGen def record `V6_vunpackoh_128B`.
  **L1692 CN**: 声明 TableGen def 记录 `V6_vunpackoh_128B`。
- **L1693 EN**: Declares TableGen def record `V6_vunpackub`.
  **L1693 CN**: 声明 TableGen def 记录 `V6_vunpackub`。
- **L1694 EN**: Declares TableGen def record `V6_vunpackub_128B`.
  **L1694 CN**: 声明 TableGen def 记录 `V6_vunpackub_128B`。
- **L1695 EN**: Declares TableGen def record `V6_vunpackuh`.
  **L1695 CN**: 声明 TableGen def 记录 `V6_vunpackuh`。
- **L1696 EN**: Declares TableGen def record `V6_vunpackuh_128B`.
  **L1696 CN**: 声明 TableGen def 记录 `V6_vunpackuh_128B`。
- **L1697 EN**: Declares TableGen def record `V6_vxor`.
  **L1697 CN**: 声明 TableGen def 记录 `V6_vxor`。
- **L1698 EN**: Declares TableGen def record `V6_vxor_128B`.
  **L1698 CN**: 声明 TableGen def 记录 `V6_vxor_128B`。
- **L1699 EN**: Declares TableGen def record `V6_vzb`.
  **L1699 CN**: 声明 TableGen def 记录 `V6_vzb`。
- **L1700 EN**: Declares TableGen def record `V6_vzb_128B`.
  **L1700 CN**: 声明 TableGen def 记录 `V6_vzb_128B`。
- **L1701 EN**: Declares TableGen def record `V6_vzh`.
  **L1701 CN**: 声明 TableGen def 记录 `V6_vzh`。
- **L1702 EN**: Declares TableGen def record `V6_vzh_128B`.
  **L1702 CN**: 声明 TableGen def 记录 `V6_vzh_128B`。
- **L1703 EN**: Closes the current lexical scope or compound statement.
  **L1703 CN**: 结束当前词法作用域或复合语句块。
- **L1704 EN**: Blank line separating nearby declarations or logic blocks.
  **L1704 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1705 EN**: Comment explains nearby logic, constraints, or intent: `V62 HVX Instructions.`.
  **L1705 CN**: 注释解释附近代码的逻辑、约束或设计意图：`V62 HVX Instructions.`。
- **L1706 EN**: Blank line separating nearby declarations or logic blocks.
  **L1706 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1707 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = HVXV62.Features in {`.
  **L1707 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = HVXV62.Features in {`。
- **L1708 EN**: Declares TableGen def record `V6_lvsplatb`.
  **L1708 CN**: 声明 TableGen def 记录 `V6_lvsplatb`。

### Lines 1709-1736

````tablegen
  def V6_lvsplatb_128B : HexagonBuiltin<"_Vector<32, int>(int)">;
  def V6_lvsplath : HexagonBuiltin<"_Vector<16, int>(int)">;
  def V6_lvsplath_128B : HexagonBuiltin<"_Vector<32, int>(int)">;
  def V6_pred_scalar2v2 : HexagonBuiltin<"_Vector<64, bool>(int)">;
  def V6_pred_scalar2v2_128B : HexagonBuiltin<"_Vector<128, bool>(int)">;
  def V6_shuffeqh : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<64, bool>)">;
  def V6_shuffeqh_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<128, bool>)">;
  def V6_shuffeqw : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<64, bool>)">;
  def V6_shuffeqw_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<128, bool>)">;
  def V6_vaddbsat : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vaddbsat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vaddbsat_dv : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vaddbsat_dv_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<64, int>)">;
  def V6_vaddcarry : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, void *)">;
  def V6_vaddcarry_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, void *)">;
  def V6_vaddclbh : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vaddclbh_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vaddclbw : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vaddclbw_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vaddhw_acc : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vaddhw_acc_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vaddubh_acc : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vaddubh_acc_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vaddububb_sat : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vaddububb_sat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vadduhw_acc : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vadduhw_acc_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vadduwsat : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
````
- **L1709 EN**: Declares TableGen def record `V6_lvsplatb_128B`.
  **L1709 CN**: 声明 TableGen def 记录 `V6_lvsplatb_128B`。
- **L1710 EN**: Declares TableGen def record `V6_lvsplath`.
  **L1710 CN**: 声明 TableGen def 记录 `V6_lvsplath`。
- **L1711 EN**: Declares TableGen def record `V6_lvsplath_128B`.
  **L1711 CN**: 声明 TableGen def 记录 `V6_lvsplath_128B`。
- **L1712 EN**: Declares TableGen def record `V6_pred_scalar2v2`.
  **L1712 CN**: 声明 TableGen def 记录 `V6_pred_scalar2v2`。
- **L1713 EN**: Declares TableGen def record `V6_pred_scalar2v2_128B`.
  **L1713 CN**: 声明 TableGen def 记录 `V6_pred_scalar2v2_128B`。
- **L1714 EN**: Declares TableGen def record `V6_shuffeqh`.
  **L1714 CN**: 声明 TableGen def 记录 `V6_shuffeqh`。
- **L1715 EN**: Declares TableGen def record `V6_shuffeqh_128B`.
  **L1715 CN**: 声明 TableGen def 记录 `V6_shuffeqh_128B`。
- **L1716 EN**: Declares TableGen def record `V6_shuffeqw`.
  **L1716 CN**: 声明 TableGen def 记录 `V6_shuffeqw`。
- **L1717 EN**: Declares TableGen def record `V6_shuffeqw_128B`.
  **L1717 CN**: 声明 TableGen def 记录 `V6_shuffeqw_128B`。
- **L1718 EN**: Declares TableGen def record `V6_vaddbsat`.
  **L1718 CN**: 声明 TableGen def 记录 `V6_vaddbsat`。
- **L1719 EN**: Declares TableGen def record `V6_vaddbsat_128B`.
  **L1719 CN**: 声明 TableGen def 记录 `V6_vaddbsat_128B`。
- **L1720 EN**: Declares TableGen def record `V6_vaddbsat_dv`.
  **L1720 CN**: 声明 TableGen def 记录 `V6_vaddbsat_dv`。
- **L1721 EN**: Declares TableGen def record `V6_vaddbsat_dv_128B`.
  **L1721 CN**: 声明 TableGen def 记录 `V6_vaddbsat_dv_128B`。
- **L1722 EN**: Declares TableGen def record `V6_vaddcarry`.
  **L1722 CN**: 声明 TableGen def 记录 `V6_vaddcarry`。
- **L1723 EN**: Declares TableGen def record `V6_vaddcarry_128B`.
  **L1723 CN**: 声明 TableGen def 记录 `V6_vaddcarry_128B`。
- **L1724 EN**: Declares TableGen def record `V6_vaddclbh`.
  **L1724 CN**: 声明 TableGen def 记录 `V6_vaddclbh`。
- **L1725 EN**: Declares TableGen def record `V6_vaddclbh_128B`.
  **L1725 CN**: 声明 TableGen def 记录 `V6_vaddclbh_128B`。
- **L1726 EN**: Declares TableGen def record `V6_vaddclbw`.
  **L1726 CN**: 声明 TableGen def 记录 `V6_vaddclbw`。
- **L1727 EN**: Declares TableGen def record `V6_vaddclbw_128B`.
  **L1727 CN**: 声明 TableGen def 记录 `V6_vaddclbw_128B`。
- **L1728 EN**: Declares TableGen def record `V6_vaddhw_acc`.
  **L1728 CN**: 声明 TableGen def 记录 `V6_vaddhw_acc`。
- **L1729 EN**: Declares TableGen def record `V6_vaddhw_acc_128B`.
  **L1729 CN**: 声明 TableGen def 记录 `V6_vaddhw_acc_128B`。
- **L1730 EN**: Declares TableGen def record `V6_vaddubh_acc`.
  **L1730 CN**: 声明 TableGen def 记录 `V6_vaddubh_acc`。
- **L1731 EN**: Declares TableGen def record `V6_vaddubh_acc_128B`.
  **L1731 CN**: 声明 TableGen def 记录 `V6_vaddubh_acc_128B`。
- **L1732 EN**: Declares TableGen def record `V6_vaddububb_sat`.
  **L1732 CN**: 声明 TableGen def 记录 `V6_vaddububb_sat`。
- **L1733 EN**: Declares TableGen def record `V6_vaddububb_sat_128B`.
  **L1733 CN**: 声明 TableGen def 记录 `V6_vaddububb_sat_128B`。
- **L1734 EN**: Declares TableGen def record `V6_vadduhw_acc`.
  **L1734 CN**: 声明 TableGen def 记录 `V6_vadduhw_acc`。
- **L1735 EN**: Declares TableGen def record `V6_vadduhw_acc_128B`.
  **L1735 CN**: 声明 TableGen def 记录 `V6_vadduhw_acc_128B`。
- **L1736 EN**: Declares TableGen def record `V6_vadduwsat`.
  **L1736 CN**: 声明 TableGen def 记录 `V6_vadduwsat`。

### Lines 1737-1764

````tablegen
  def V6_vadduwsat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vadduwsat_dv : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vadduwsat_dv_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<64, int>)">;
  def V6_vandnqrt : HexagonBuiltin<"_Vector<16, int>(_Vector<64, bool>, int)">;
  def V6_vandnqrt_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<128, bool>, int)">;
  def V6_vandnqrt_acc : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<64, bool>, int)">;
  def V6_vandnqrt_acc_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<128, bool>, int)">;
  def V6_vandvnqv : HexagonBuiltin<"_Vector<16, int>(_Vector<64, bool>, _Vector<16, int>)">;
  def V6_vandvnqv_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<128, bool>, _Vector<32, int>)">;
  def V6_vandvqv : HexagonBuiltin<"_Vector<16, int>(_Vector<64, bool>, _Vector<16, int>)">;
  def V6_vandvqv_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<128, bool>, _Vector<32, int>)">;
  def V6_vasrhbsat : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, int)">;
  def V6_vasrhbsat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vasruwuhrndsat : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, int)">;
  def V6_vasruwuhrndsat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vasrwuhrndsat : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, int)">;
  def V6_vasrwuhrndsat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vlsrb : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, int)">;
  def V6_vlsrb_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, int)">;
  def V6_vlutvvb_nm : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, int)">;
  def V6_vlutvvb_nm_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vlutvvb_oracci : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, _Vector<16, int>, unsigned _Constant int)">;
  def V6_vlutvvb_oracci_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, _Vector<32, int>, unsigned _Constant int)">;
  def V6_vlutvvbi : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, unsigned _Constant int)">;
  def V6_vlutvvbi_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, unsigned _Constant int)">;
  def V6_vlutvwh_nm : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>, _Vector<16, int>, int)">;
  def V6_vlutvwh_nm_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vlutvwh_oracci : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<16, int>, _Vector<16, int>, unsigned _Constant int)">;
````
- **L1737 EN**: Declares TableGen def record `V6_vadduwsat_128B`.
  **L1737 CN**: 声明 TableGen def 记录 `V6_vadduwsat_128B`。
- **L1738 EN**: Declares TableGen def record `V6_vadduwsat_dv`.
  **L1738 CN**: 声明 TableGen def 记录 `V6_vadduwsat_dv`。
- **L1739 EN**: Declares TableGen def record `V6_vadduwsat_dv_128B`.
  **L1739 CN**: 声明 TableGen def 记录 `V6_vadduwsat_dv_128B`。
- **L1740 EN**: Declares TableGen def record `V6_vandnqrt`.
  **L1740 CN**: 声明 TableGen def 记录 `V6_vandnqrt`。
- **L1741 EN**: Declares TableGen def record `V6_vandnqrt_128B`.
  **L1741 CN**: 声明 TableGen def 记录 `V6_vandnqrt_128B`。
- **L1742 EN**: Declares TableGen def record `V6_vandnqrt_acc`.
  **L1742 CN**: 声明 TableGen def 记录 `V6_vandnqrt_acc`。
- **L1743 EN**: Declares TableGen def record `V6_vandnqrt_acc_128B`.
  **L1743 CN**: 声明 TableGen def 记录 `V6_vandnqrt_acc_128B`。
- **L1744 EN**: Declares TableGen def record `V6_vandvnqv`.
  **L1744 CN**: 声明 TableGen def 记录 `V6_vandvnqv`。
- **L1745 EN**: Declares TableGen def record `V6_vandvnqv_128B`.
  **L1745 CN**: 声明 TableGen def 记录 `V6_vandvnqv_128B`。
- **L1746 EN**: Declares TableGen def record `V6_vandvqv`.
  **L1746 CN**: 声明 TableGen def 记录 `V6_vandvqv`。
- **L1747 EN**: Declares TableGen def record `V6_vandvqv_128B`.
  **L1747 CN**: 声明 TableGen def 记录 `V6_vandvqv_128B`。
- **L1748 EN**: Declares TableGen def record `V6_vasrhbsat`.
  **L1748 CN**: 声明 TableGen def 记录 `V6_vasrhbsat`。
- **L1749 EN**: Declares TableGen def record `V6_vasrhbsat_128B`.
  **L1749 CN**: 声明 TableGen def 记录 `V6_vasrhbsat_128B`。
- **L1750 EN**: Declares TableGen def record `V6_vasruwuhrndsat`.
  **L1750 CN**: 声明 TableGen def 记录 `V6_vasruwuhrndsat`。
- **L1751 EN**: Declares TableGen def record `V6_vasruwuhrndsat_128B`.
  **L1751 CN**: 声明 TableGen def 记录 `V6_vasruwuhrndsat_128B`。
- **L1752 EN**: Declares TableGen def record `V6_vasrwuhrndsat`.
  **L1752 CN**: 声明 TableGen def 记录 `V6_vasrwuhrndsat`。
- **L1753 EN**: Declares TableGen def record `V6_vasrwuhrndsat_128B`.
  **L1753 CN**: 声明 TableGen def 记录 `V6_vasrwuhrndsat_128B`。
- **L1754 EN**: Declares TableGen def record `V6_vlsrb`.
  **L1754 CN**: 声明 TableGen def 记录 `V6_vlsrb`。
- **L1755 EN**: Declares TableGen def record `V6_vlsrb_128B`.
  **L1755 CN**: 声明 TableGen def 记录 `V6_vlsrb_128B`。
- **L1756 EN**: Declares TableGen def record `V6_vlutvvb_nm`.
  **L1756 CN**: 声明 TableGen def 记录 `V6_vlutvvb_nm`。
- **L1757 EN**: Declares TableGen def record `V6_vlutvvb_nm_128B`.
  **L1757 CN**: 声明 TableGen def 记录 `V6_vlutvvb_nm_128B`。
- **L1758 EN**: Declares TableGen def record `V6_vlutvvb_oracci`.
  **L1758 CN**: 声明 TableGen def 记录 `V6_vlutvvb_oracci`。
- **L1759 EN**: Declares TableGen def record `V6_vlutvvb_oracci_128B`.
  **L1759 CN**: 声明 TableGen def 记录 `V6_vlutvvb_oracci_128B`。
- **L1760 EN**: Declares TableGen def record `V6_vlutvvbi`.
  **L1760 CN**: 声明 TableGen def 记录 `V6_vlutvvbi`。
- **L1761 EN**: Declares TableGen def record `V6_vlutvvbi_128B`.
  **L1761 CN**: 声明 TableGen def 记录 `V6_vlutvvbi_128B`。
- **L1762 EN**: Declares TableGen def record `V6_vlutvwh_nm`.
  **L1762 CN**: 声明 TableGen def 记录 `V6_vlutvwh_nm`。
- **L1763 EN**: Declares TableGen def record `V6_vlutvwh_nm_128B`.
  **L1763 CN**: 声明 TableGen def 记录 `V6_vlutvwh_nm_128B`。
- **L1764 EN**: Declares TableGen def record `V6_vlutvwh_oracci`.
  **L1764 CN**: 声明 TableGen def 记录 `V6_vlutvwh_oracci`。

### Lines 1765-1792

````tablegen
  def V6_vlutvwh_oracci_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<32, int>, _Vector<32, int>, unsigned _Constant int)">;
  def V6_vlutvwhi : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>, _Vector<16, int>, unsigned _Constant int)">;
  def V6_vlutvwhi_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>, _Vector<32, int>, unsigned _Constant int)">;
  def V6_vmaxb : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vmaxb_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vminb : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vminb_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpauhb : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, int)">;
  def V6_vmpauhb_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, int)">;
  def V6_vmpauhb_acc : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vmpauhb_acc_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<64, int>, int)">;
  def V6_vmpyewuh_64 : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpyewuh_64_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpyiwub : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, int)">;
  def V6_vmpyiwub_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, int)">;
  def V6_vmpyiwub_acc : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, int)">;
  def V6_vmpyiwub_acc_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vmpyowh_64_acc : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpyowh_64_acc_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vrounduhub : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vrounduhub_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vrounduwuh : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vrounduwuh_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vsatuwuh : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vsatuwuh_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vsubbsat : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vsubbsat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vsubbsat_dv : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
````
- **L1765 EN**: Declares TableGen def record `V6_vlutvwh_oracci_128B`.
  **L1765 CN**: 声明 TableGen def 记录 `V6_vlutvwh_oracci_128B`。
- **L1766 EN**: Declares TableGen def record `V6_vlutvwhi`.
  **L1766 CN**: 声明 TableGen def 记录 `V6_vlutvwhi`。
- **L1767 EN**: Declares TableGen def record `V6_vlutvwhi_128B`.
  **L1767 CN**: 声明 TableGen def 记录 `V6_vlutvwhi_128B`。
- **L1768 EN**: Declares TableGen def record `V6_vmaxb`.
  **L1768 CN**: 声明 TableGen def 记录 `V6_vmaxb`。
- **L1769 EN**: Declares TableGen def record `V6_vmaxb_128B`.
  **L1769 CN**: 声明 TableGen def 记录 `V6_vmaxb_128B`。
- **L1770 EN**: Declares TableGen def record `V6_vminb`.
  **L1770 CN**: 声明 TableGen def 记录 `V6_vminb`。
- **L1771 EN**: Declares TableGen def record `V6_vminb_128B`.
  **L1771 CN**: 声明 TableGen def 记录 `V6_vminb_128B`。
- **L1772 EN**: Declares TableGen def record `V6_vmpauhb`.
  **L1772 CN**: 声明 TableGen def 记录 `V6_vmpauhb`。
- **L1773 EN**: Declares TableGen def record `V6_vmpauhb_128B`.
  **L1773 CN**: 声明 TableGen def 记录 `V6_vmpauhb_128B`。
- **L1774 EN**: Declares TableGen def record `V6_vmpauhb_acc`.
  **L1774 CN**: 声明 TableGen def 记录 `V6_vmpauhb_acc`。
- **L1775 EN**: Declares TableGen def record `V6_vmpauhb_acc_128B`.
  **L1775 CN**: 声明 TableGen def 记录 `V6_vmpauhb_acc_128B`。
- **L1776 EN**: Declares TableGen def record `V6_vmpyewuh_64`.
  **L1776 CN**: 声明 TableGen def 记录 `V6_vmpyewuh_64`。
- **L1777 EN**: Declares TableGen def record `V6_vmpyewuh_64_128B`.
  **L1777 CN**: 声明 TableGen def 记录 `V6_vmpyewuh_64_128B`。
- **L1778 EN**: Declares TableGen def record `V6_vmpyiwub`.
  **L1778 CN**: 声明 TableGen def 记录 `V6_vmpyiwub`。
- **L1779 EN**: Declares TableGen def record `V6_vmpyiwub_128B`.
  **L1779 CN**: 声明 TableGen def 记录 `V6_vmpyiwub_128B`。
- **L1780 EN**: Declares TableGen def record `V6_vmpyiwub_acc`.
  **L1780 CN**: 声明 TableGen def 记录 `V6_vmpyiwub_acc`。
- **L1781 EN**: Declares TableGen def record `V6_vmpyiwub_acc_128B`.
  **L1781 CN**: 声明 TableGen def 记录 `V6_vmpyiwub_acc_128B`。
- **L1782 EN**: Declares TableGen def record `V6_vmpyowh_64_acc`.
  **L1782 CN**: 声明 TableGen def 记录 `V6_vmpyowh_64_acc`。
- **L1783 EN**: Declares TableGen def record `V6_vmpyowh_64_acc_128B`.
  **L1783 CN**: 声明 TableGen def 记录 `V6_vmpyowh_64_acc_128B`。
- **L1784 EN**: Declares TableGen def record `V6_vrounduhub`.
  **L1784 CN**: 声明 TableGen def 记录 `V6_vrounduhub`。
- **L1785 EN**: Declares TableGen def record `V6_vrounduhub_128B`.
  **L1785 CN**: 声明 TableGen def 记录 `V6_vrounduhub_128B`。
- **L1786 EN**: Declares TableGen def record `V6_vrounduwuh`.
  **L1786 CN**: 声明 TableGen def 记录 `V6_vrounduwuh`。
- **L1787 EN**: Declares TableGen def record `V6_vrounduwuh_128B`.
  **L1787 CN**: 声明 TableGen def 记录 `V6_vrounduwuh_128B`。
- **L1788 EN**: Declares TableGen def record `V6_vsatuwuh`.
  **L1788 CN**: 声明 TableGen def 记录 `V6_vsatuwuh`。
- **L1789 EN**: Declares TableGen def record `V6_vsatuwuh_128B`.
  **L1789 CN**: 声明 TableGen def 记录 `V6_vsatuwuh_128B`。
- **L1790 EN**: Declares TableGen def record `V6_vsubbsat`.
  **L1790 CN**: 声明 TableGen def 记录 `V6_vsubbsat`。
- **L1791 EN**: Declares TableGen def record `V6_vsubbsat_128B`.
  **L1791 CN**: 声明 TableGen def 记录 `V6_vsubbsat_128B`。
- **L1792 EN**: Declares TableGen def record `V6_vsubbsat_dv`.
  **L1792 CN**: 声明 TableGen def 记录 `V6_vsubbsat_dv`。

### Lines 1793-1820

````tablegen
  def V6_vsubbsat_dv_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<64, int>)">;
  def V6_vsubcarry : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, void *)">;
  def V6_vsubcarry_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, void *)">;
  def V6_vsubububb_sat : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vsubububb_sat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vsubuwsat : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vsubuwsat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vsubuwsat_dv : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vsubuwsat_dv_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<64, int>)">;
}

// V65 HVX Instructions.

let Features = HVXV65.Features in {
  def V6_vabsb : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vabsb_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vabsb_sat : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vabsb_sat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vaslh_acc : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, int)">;
  def V6_vaslh_acc_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vasrh_acc : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, int)">;
  def V6_vasrh_acc_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vasruhubrndsat : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, int)">;
  def V6_vasruhubrndsat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vasruhubsat : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, int)">;
  def V6_vasruhubsat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vasruwuhsat : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, int)">;
  def V6_vasruwuhsat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
````
- **L1793 EN**: Declares TableGen def record `V6_vsubbsat_dv_128B`.
  **L1793 CN**: 声明 TableGen def 记录 `V6_vsubbsat_dv_128B`。
- **L1794 EN**: Declares TableGen def record `V6_vsubcarry`.
  **L1794 CN**: 声明 TableGen def 记录 `V6_vsubcarry`。
- **L1795 EN**: Declares TableGen def record `V6_vsubcarry_128B`.
  **L1795 CN**: 声明 TableGen def 记录 `V6_vsubcarry_128B`。
- **L1796 EN**: Declares TableGen def record `V6_vsubububb_sat`.
  **L1796 CN**: 声明 TableGen def 记录 `V6_vsubububb_sat`。
- **L1797 EN**: Declares TableGen def record `V6_vsubububb_sat_128B`.
  **L1797 CN**: 声明 TableGen def 记录 `V6_vsubububb_sat_128B`。
- **L1798 EN**: Declares TableGen def record `V6_vsubuwsat`.
  **L1798 CN**: 声明 TableGen def 记录 `V6_vsubuwsat`。
- **L1799 EN**: Declares TableGen def record `V6_vsubuwsat_128B`.
  **L1799 CN**: 声明 TableGen def 记录 `V6_vsubuwsat_128B`。
- **L1800 EN**: Declares TableGen def record `V6_vsubuwsat_dv`.
  **L1800 CN**: 声明 TableGen def 记录 `V6_vsubuwsat_dv`。
- **L1801 EN**: Declares TableGen def record `V6_vsubuwsat_dv_128B`.
  **L1801 CN**: 声明 TableGen def 记录 `V6_vsubuwsat_dv_128B`。
- **L1802 EN**: Closes the current lexical scope or compound statement.
  **L1802 CN**: 结束当前词法作用域或复合语句块。
- **L1803 EN**: Blank line separating nearby declarations or logic blocks.
  **L1803 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1804 EN**: Comment explains nearby logic, constraints, or intent: `V65 HVX Instructions.`.
  **L1804 CN**: 注释解释附近代码的逻辑、约束或设计意图：`V65 HVX Instructions.`。
- **L1805 EN**: Blank line separating nearby declarations or logic blocks.
  **L1805 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1806 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = HVXV65.Features in {`.
  **L1806 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = HVXV65.Features in {`。
- **L1807 EN**: Declares TableGen def record `V6_vabsb`.
  **L1807 CN**: 声明 TableGen def 记录 `V6_vabsb`。
- **L1808 EN**: Declares TableGen def record `V6_vabsb_128B`.
  **L1808 CN**: 声明 TableGen def 记录 `V6_vabsb_128B`。
- **L1809 EN**: Declares TableGen def record `V6_vabsb_sat`.
  **L1809 CN**: 声明 TableGen def 记录 `V6_vabsb_sat`。
- **L1810 EN**: Declares TableGen def record `V6_vabsb_sat_128B`.
  **L1810 CN**: 声明 TableGen def 记录 `V6_vabsb_sat_128B`。
- **L1811 EN**: Declares TableGen def record `V6_vaslh_acc`.
  **L1811 CN**: 声明 TableGen def 记录 `V6_vaslh_acc`。
- **L1812 EN**: Declares TableGen def record `V6_vaslh_acc_128B`.
  **L1812 CN**: 声明 TableGen def 记录 `V6_vaslh_acc_128B`。
- **L1813 EN**: Declares TableGen def record `V6_vasrh_acc`.
  **L1813 CN**: 声明 TableGen def 记录 `V6_vasrh_acc`。
- **L1814 EN**: Declares TableGen def record `V6_vasrh_acc_128B`.
  **L1814 CN**: 声明 TableGen def 记录 `V6_vasrh_acc_128B`。
- **L1815 EN**: Declares TableGen def record `V6_vasruhubrndsat`.
  **L1815 CN**: 声明 TableGen def 记录 `V6_vasruhubrndsat`。
- **L1816 EN**: Declares TableGen def record `V6_vasruhubrndsat_128B`.
  **L1816 CN**: 声明 TableGen def 记录 `V6_vasruhubrndsat_128B`。
- **L1817 EN**: Declares TableGen def record `V6_vasruhubsat`.
  **L1817 CN**: 声明 TableGen def 记录 `V6_vasruhubsat`。
- **L1818 EN**: Declares TableGen def record `V6_vasruhubsat_128B`.
  **L1818 CN**: 声明 TableGen def 记录 `V6_vasruhubsat_128B`。
- **L1819 EN**: Declares TableGen def record `V6_vasruwuhsat`.
  **L1819 CN**: 声明 TableGen def 记录 `V6_vasruwuhsat`。
- **L1820 EN**: Declares TableGen def record `V6_vasruwuhsat_128B`.
  **L1820 CN**: 声明 TableGen def 记录 `V6_vasruwuhsat_128B`。

### Lines 1821-1848

````tablegen
  def V6_vavgb : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vavgb_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vavgbrnd : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vavgbrnd_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vavguw : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vavguw_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vavguwrnd : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vavguwrnd_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vdd0 : HexagonBuiltin<"_Vector<32, int>()">;
  def V6_vdd0_128B : HexagonBuiltin<"_Vector<64, int>()">;
  def V6_vgathermh : HexagonBuiltin<"void(void *, int, int, _Vector<16, int>)">;
  def V6_vgathermh_128B : HexagonBuiltin<"void(void *, int, int, _Vector<32, int>)">;
  def V6_vgathermhq : HexagonBuiltin<"void(void *, _Vector<64, bool>, int, int, _Vector<16, int>)">;
  def V6_vgathermhq_128B : HexagonBuiltin<"void(void *, _Vector<128, bool>, int, int, _Vector<32, int>)">;
  def V6_vgathermhw : HexagonBuiltin<"void(void *, int, int, _Vector<32, int>)">;
  def V6_vgathermhw_128B : HexagonBuiltin<"void(void *, int, int, _Vector<64, int>)">;
  def V6_vgathermhwq : HexagonBuiltin<"void(void *, _Vector<64, bool>, int, int, _Vector<32, int>)">;
  def V6_vgathermhwq_128B : HexagonBuiltin<"void(void *, _Vector<128, bool>, int, int, _Vector<64, int>)">;
  def V6_vgathermw : HexagonBuiltin<"void(void *, int, int, _Vector<16, int>)">;
  def V6_vgathermw_128B : HexagonBuiltin<"void(void *, int, int, _Vector<32, int>)">;
  def V6_vgathermwq : HexagonBuiltin<"void(void *, _Vector<64, bool>, int, int, _Vector<16, int>)">;
  def V6_vgathermwq_128B : HexagonBuiltin<"void(void *, _Vector<128, bool>, int, int, _Vector<32, int>)">;
  def V6_vlut4 : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, long long int)">;
  def V6_vlut4_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, long long int)">;
  def V6_vmpabuu : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, int)">;
  def V6_vmpabuu_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, int)">;
  def V6_vmpabuu_acc : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vmpabuu_acc_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<64, int>, int)">;
````
- **L1821 EN**: Declares TableGen def record `V6_vavgb`.
  **L1821 CN**: 声明 TableGen def 记录 `V6_vavgb`。
- **L1822 EN**: Declares TableGen def record `V6_vavgb_128B`.
  **L1822 CN**: 声明 TableGen def 记录 `V6_vavgb_128B`。
- **L1823 EN**: Declares TableGen def record `V6_vavgbrnd`.
  **L1823 CN**: 声明 TableGen def 记录 `V6_vavgbrnd`。
- **L1824 EN**: Declares TableGen def record `V6_vavgbrnd_128B`.
  **L1824 CN**: 声明 TableGen def 记录 `V6_vavgbrnd_128B`。
- **L1825 EN**: Declares TableGen def record `V6_vavguw`.
  **L1825 CN**: 声明 TableGen def 记录 `V6_vavguw`。
- **L1826 EN**: Declares TableGen def record `V6_vavguw_128B`.
  **L1826 CN**: 声明 TableGen def 记录 `V6_vavguw_128B`。
- **L1827 EN**: Declares TableGen def record `V6_vavguwrnd`.
  **L1827 CN**: 声明 TableGen def 记录 `V6_vavguwrnd`。
- **L1828 EN**: Declares TableGen def record `V6_vavguwrnd_128B`.
  **L1828 CN**: 声明 TableGen def 记录 `V6_vavguwrnd_128B`。
- **L1829 EN**: Declares TableGen def record `V6_vdd0`.
  **L1829 CN**: 声明 TableGen def 记录 `V6_vdd0`。
- **L1830 EN**: Declares TableGen def record `V6_vdd0_128B`.
  **L1830 CN**: 声明 TableGen def 记录 `V6_vdd0_128B`。
- **L1831 EN**: Declares TableGen def record `V6_vgathermh`.
  **L1831 CN**: 声明 TableGen def 记录 `V6_vgathermh`。
- **L1832 EN**: Declares TableGen def record `V6_vgathermh_128B`.
  **L1832 CN**: 声明 TableGen def 记录 `V6_vgathermh_128B`。
- **L1833 EN**: Declares TableGen def record `V6_vgathermhq`.
  **L1833 CN**: 声明 TableGen def 记录 `V6_vgathermhq`。
- **L1834 EN**: Declares TableGen def record `V6_vgathermhq_128B`.
  **L1834 CN**: 声明 TableGen def 记录 `V6_vgathermhq_128B`。
- **L1835 EN**: Declares TableGen def record `V6_vgathermhw`.
  **L1835 CN**: 声明 TableGen def 记录 `V6_vgathermhw`。
- **L1836 EN**: Declares TableGen def record `V6_vgathermhw_128B`.
  **L1836 CN**: 声明 TableGen def 记录 `V6_vgathermhw_128B`。
- **L1837 EN**: Declares TableGen def record `V6_vgathermhwq`.
  **L1837 CN**: 声明 TableGen def 记录 `V6_vgathermhwq`。
- **L1838 EN**: Declares TableGen def record `V6_vgathermhwq_128B`.
  **L1838 CN**: 声明 TableGen def 记录 `V6_vgathermhwq_128B`。
- **L1839 EN**: Declares TableGen def record `V6_vgathermw`.
  **L1839 CN**: 声明 TableGen def 记录 `V6_vgathermw`。
- **L1840 EN**: Declares TableGen def record `V6_vgathermw_128B`.
  **L1840 CN**: 声明 TableGen def 记录 `V6_vgathermw_128B`。
- **L1841 EN**: Declares TableGen def record `V6_vgathermwq`.
  **L1841 CN**: 声明 TableGen def 记录 `V6_vgathermwq`。
- **L1842 EN**: Declares TableGen def record `V6_vgathermwq_128B`.
  **L1842 CN**: 声明 TableGen def 记录 `V6_vgathermwq_128B`。
- **L1843 EN**: Declares TableGen def record `V6_vlut4`.
  **L1843 CN**: 声明 TableGen def 记录 `V6_vlut4`。
- **L1844 EN**: Declares TableGen def record `V6_vlut4_128B`.
  **L1844 CN**: 声明 TableGen def 记录 `V6_vlut4_128B`。
- **L1845 EN**: Declares TableGen def record `V6_vmpabuu`.
  **L1845 CN**: 声明 TableGen def 记录 `V6_vmpabuu`。
- **L1846 EN**: Declares TableGen def record `V6_vmpabuu_128B`.
  **L1846 CN**: 声明 TableGen def 记录 `V6_vmpabuu_128B`。
- **L1847 EN**: Declares TableGen def record `V6_vmpabuu_acc`.
  **L1847 CN**: 声明 TableGen def 记录 `V6_vmpabuu_acc`。
- **L1848 EN**: Declares TableGen def record `V6_vmpabuu_acc_128B`.
  **L1848 CN**: 声明 TableGen def 记录 `V6_vmpabuu_acc_128B`。

### Lines 1849-1876

````tablegen
  def V6_vmpahhsat : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, long long int)">;
  def V6_vmpahhsat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, long long int)">;
  def V6_vmpauhuhsat : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, long long int)">;
  def V6_vmpauhuhsat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, long long int)">;
  def V6_vmpsuhuhsat : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, long long int)">;
  def V6_vmpsuhuhsat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, long long int)">;
  def V6_vmpyh_acc : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<16, int>, int)">;
  def V6_vmpyh_acc_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<32, int>, int)">;
  def V6_vmpyuhe : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, int)">;
  def V6_vmpyuhe_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, int)">;
  def V6_vmpyuhe_acc : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, int)">;
  def V6_vmpyuhe_acc_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vnavgb : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vnavgb_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vprefixqb : HexagonBuiltin<"_Vector<16, int>(_Vector<64, bool>)">;
  def V6_vprefixqb_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<128, bool>)">;
  def V6_vprefixqh : HexagonBuiltin<"_Vector<16, int>(_Vector<64, bool>)">;
  def V6_vprefixqh_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<128, bool>)">;
  def V6_vprefixqw : HexagonBuiltin<"_Vector<16, int>(_Vector<64, bool>)">;
  def V6_vprefixqw_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<128, bool>)">;
  def V6_vscattermh : HexagonBuiltin<"void(int, int, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vscattermh_128B : HexagonBuiltin<"void(int, int, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vscattermh_add : HexagonBuiltin<"void(int, int, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vscattermh_add_128B : HexagonBuiltin<"void(int, int, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vscattermhq : HexagonBuiltin<"void(_Vector<64, bool>, int, int, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vscattermhq_128B : HexagonBuiltin<"void(_Vector<128, bool>, int, int, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vscattermhw : HexagonBuiltin<"void(int, int, _Vector<32, int>, _Vector<16, int>)">;
  def V6_vscattermhw_128B : HexagonBuiltin<"void(int, int, _Vector<64, int>, _Vector<32, int>)">;
````
- **L1849 EN**: Declares TableGen def record `V6_vmpahhsat`.
  **L1849 CN**: 声明 TableGen def 记录 `V6_vmpahhsat`。
- **L1850 EN**: Declares TableGen def record `V6_vmpahhsat_128B`.
  **L1850 CN**: 声明 TableGen def 记录 `V6_vmpahhsat_128B`。
- **L1851 EN**: Declares TableGen def record `V6_vmpauhuhsat`.
  **L1851 CN**: 声明 TableGen def 记录 `V6_vmpauhuhsat`。
- **L1852 EN**: Declares TableGen def record `V6_vmpauhuhsat_128B`.
  **L1852 CN**: 声明 TableGen def 记录 `V6_vmpauhuhsat_128B`。
- **L1853 EN**: Declares TableGen def record `V6_vmpsuhuhsat`.
  **L1853 CN**: 声明 TableGen def 记录 `V6_vmpsuhuhsat`。
- **L1854 EN**: Declares TableGen def record `V6_vmpsuhuhsat_128B`.
  **L1854 CN**: 声明 TableGen def 记录 `V6_vmpsuhuhsat_128B`。
- **L1855 EN**: Declares TableGen def record `V6_vmpyh_acc`.
  **L1855 CN**: 声明 TableGen def 记录 `V6_vmpyh_acc`。
- **L1856 EN**: Declares TableGen def record `V6_vmpyh_acc_128B`.
  **L1856 CN**: 声明 TableGen def 记录 `V6_vmpyh_acc_128B`。
- **L1857 EN**: Declares TableGen def record `V6_vmpyuhe`.
  **L1857 CN**: 声明 TableGen def 记录 `V6_vmpyuhe`。
- **L1858 EN**: Declares TableGen def record `V6_vmpyuhe_128B`.
  **L1858 CN**: 声明 TableGen def 记录 `V6_vmpyuhe_128B`。
- **L1859 EN**: Declares TableGen def record `V6_vmpyuhe_acc`.
  **L1859 CN**: 声明 TableGen def 记录 `V6_vmpyuhe_acc`。
- **L1860 EN**: Declares TableGen def record `V6_vmpyuhe_acc_128B`.
  **L1860 CN**: 声明 TableGen def 记录 `V6_vmpyuhe_acc_128B`。
- **L1861 EN**: Declares TableGen def record `V6_vnavgb`.
  **L1861 CN**: 声明 TableGen def 记录 `V6_vnavgb`。
- **L1862 EN**: Declares TableGen def record `V6_vnavgb_128B`.
  **L1862 CN**: 声明 TableGen def 记录 `V6_vnavgb_128B`。
- **L1863 EN**: Declares TableGen def record `V6_vprefixqb`.
  **L1863 CN**: 声明 TableGen def 记录 `V6_vprefixqb`。
- **L1864 EN**: Declares TableGen def record `V6_vprefixqb_128B`.
  **L1864 CN**: 声明 TableGen def 记录 `V6_vprefixqb_128B`。
- **L1865 EN**: Declares TableGen def record `V6_vprefixqh`.
  **L1865 CN**: 声明 TableGen def 记录 `V6_vprefixqh`。
- **L1866 EN**: Declares TableGen def record `V6_vprefixqh_128B`.
  **L1866 CN**: 声明 TableGen def 记录 `V6_vprefixqh_128B`。
- **L1867 EN**: Declares TableGen def record `V6_vprefixqw`.
  **L1867 CN**: 声明 TableGen def 记录 `V6_vprefixqw`。
- **L1868 EN**: Declares TableGen def record `V6_vprefixqw_128B`.
  **L1868 CN**: 声明 TableGen def 记录 `V6_vprefixqw_128B`。
- **L1869 EN**: Declares TableGen def record `V6_vscattermh`.
  **L1869 CN**: 声明 TableGen def 记录 `V6_vscattermh`。
- **L1870 EN**: Declares TableGen def record `V6_vscattermh_128B`.
  **L1870 CN**: 声明 TableGen def 记录 `V6_vscattermh_128B`。
- **L1871 EN**: Declares TableGen def record `V6_vscattermh_add`.
  **L1871 CN**: 声明 TableGen def 记录 `V6_vscattermh_add`。
- **L1872 EN**: Declares TableGen def record `V6_vscattermh_add_128B`.
  **L1872 CN**: 声明 TableGen def 记录 `V6_vscattermh_add_128B`。
- **L1873 EN**: Declares TableGen def record `V6_vscattermhq`.
  **L1873 CN**: 声明 TableGen def 记录 `V6_vscattermhq`。
- **L1874 EN**: Declares TableGen def record `V6_vscattermhq_128B`.
  **L1874 CN**: 声明 TableGen def 记录 `V6_vscattermhq_128B`。
- **L1875 EN**: Declares TableGen def record `V6_vscattermhw`.
  **L1875 CN**: 声明 TableGen def 记录 `V6_vscattermhw`。
- **L1876 EN**: Declares TableGen def record `V6_vscattermhw_128B`.
  **L1876 CN**: 声明 TableGen def 记录 `V6_vscattermhw_128B`。

### Lines 1877-1904

````tablegen
  def V6_vscattermhw_add : HexagonBuiltin<"void(int, int, _Vector<32, int>, _Vector<16, int>)">;
  def V6_vscattermhw_add_128B : HexagonBuiltin<"void(int, int, _Vector<64, int>, _Vector<32, int>)">;
  def V6_vscattermhwq : HexagonBuiltin<"void(_Vector<64, bool>, int, int, _Vector<32, int>, _Vector<16, int>)">;
  def V6_vscattermhwq_128B : HexagonBuiltin<"void(_Vector<128, bool>, int, int, _Vector<64, int>, _Vector<32, int>)">;
  def V6_vscattermw : HexagonBuiltin<"void(int, int, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vscattermw_128B : HexagonBuiltin<"void(int, int, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vscattermw_add : HexagonBuiltin<"void(int, int, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vscattermw_add_128B : HexagonBuiltin<"void(int, int, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vscattermwq : HexagonBuiltin<"void(_Vector<64, bool>, int, int, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vscattermwq_128B : HexagonBuiltin<"void(_Vector<128, bool>, int, int, _Vector<32, int>, _Vector<32, int>)">;
}

// V66 HVX Instructions.

let Features = HVXV66.Features in {
  def V6_vaddcarryo : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, void *)">;
  def V6_vaddcarryo_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, void *)">;
  def V6_vaddcarrysat : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, _Vector<64, bool>)">;
  def V6_vaddcarrysat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, _Vector<128, bool>)">;
  def V6_vasr_into : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vasr_into_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vrotr : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vrotr_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vsatdw : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vsatdw_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vsubcarryo : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, void *)">;
  def V6_vsubcarryo_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, void *)">;
}
````
- **L1877 EN**: Declares TableGen def record `V6_vscattermhw_add`.
  **L1877 CN**: 声明 TableGen def 记录 `V6_vscattermhw_add`。
- **L1878 EN**: Declares TableGen def record `V6_vscattermhw_add_128B`.
  **L1878 CN**: 声明 TableGen def 记录 `V6_vscattermhw_add_128B`。
- **L1879 EN**: Declares TableGen def record `V6_vscattermhwq`.
  **L1879 CN**: 声明 TableGen def 记录 `V6_vscattermhwq`。
- **L1880 EN**: Declares TableGen def record `V6_vscattermhwq_128B`.
  **L1880 CN**: 声明 TableGen def 记录 `V6_vscattermhwq_128B`。
- **L1881 EN**: Declares TableGen def record `V6_vscattermw`.
  **L1881 CN**: 声明 TableGen def 记录 `V6_vscattermw`。
- **L1882 EN**: Declares TableGen def record `V6_vscattermw_128B`.
  **L1882 CN**: 声明 TableGen def 记录 `V6_vscattermw_128B`。
- **L1883 EN**: Declares TableGen def record `V6_vscattermw_add`.
  **L1883 CN**: 声明 TableGen def 记录 `V6_vscattermw_add`。
- **L1884 EN**: Declares TableGen def record `V6_vscattermw_add_128B`.
  **L1884 CN**: 声明 TableGen def 记录 `V6_vscattermw_add_128B`。
- **L1885 EN**: Declares TableGen def record `V6_vscattermwq`.
  **L1885 CN**: 声明 TableGen def 记录 `V6_vscattermwq`。
- **L1886 EN**: Declares TableGen def record `V6_vscattermwq_128B`.
  **L1886 CN**: 声明 TableGen def 记录 `V6_vscattermwq_128B`。
- **L1887 EN**: Closes the current lexical scope or compound statement.
  **L1887 CN**: 结束当前词法作用域或复合语句块。
- **L1888 EN**: Blank line separating nearby declarations or logic blocks.
  **L1888 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1889 EN**: Comment explains nearby logic, constraints, or intent: `V66 HVX Instructions.`.
  **L1889 CN**: 注释解释附近代码的逻辑、约束或设计意图：`V66 HVX Instructions.`。
- **L1890 EN**: Blank line separating nearby declarations or logic blocks.
  **L1890 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1891 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = HVXV66.Features in {`.
  **L1891 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = HVXV66.Features in {`。
- **L1892 EN**: Declares TableGen def record `V6_vaddcarryo`.
  **L1892 CN**: 声明 TableGen def 记录 `V6_vaddcarryo`。
- **L1893 EN**: Declares TableGen def record `V6_vaddcarryo_128B`.
  **L1893 CN**: 声明 TableGen def 记录 `V6_vaddcarryo_128B`。
- **L1894 EN**: Declares TableGen def record `V6_vaddcarrysat`.
  **L1894 CN**: 声明 TableGen def 记录 `V6_vaddcarrysat`。
- **L1895 EN**: Declares TableGen def record `V6_vaddcarrysat_128B`.
  **L1895 CN**: 声明 TableGen def 记录 `V6_vaddcarrysat_128B`。
- **L1896 EN**: Declares TableGen def record `V6_vasr_into`.
  **L1896 CN**: 声明 TableGen def 记录 `V6_vasr_into`。
- **L1897 EN**: Declares TableGen def record `V6_vasr_into_128B`.
  **L1897 CN**: 声明 TableGen def 记录 `V6_vasr_into_128B`。
- **L1898 EN**: Declares TableGen def record `V6_vrotr`.
  **L1898 CN**: 声明 TableGen def 记录 `V6_vrotr`。
- **L1899 EN**: Declares TableGen def record `V6_vrotr_128B`.
  **L1899 CN**: 声明 TableGen def 记录 `V6_vrotr_128B`。
- **L1900 EN**: Declares TableGen def record `V6_vsatdw`.
  **L1900 CN**: 声明 TableGen def 记录 `V6_vsatdw`。
- **L1901 EN**: Declares TableGen def record `V6_vsatdw_128B`.
  **L1901 CN**: 声明 TableGen def 记录 `V6_vsatdw_128B`。
- **L1902 EN**: Declares TableGen def record `V6_vsubcarryo`.
  **L1902 CN**: 声明 TableGen def 记录 `V6_vsubcarryo`。
- **L1903 EN**: Declares TableGen def record `V6_vsubcarryo_128B`.
  **L1903 CN**: 声明 TableGen def 记录 `V6_vsubcarryo_128B`。
- **L1904 EN**: Closes the current lexical scope or compound statement.
  **L1904 CN**: 结束当前词法作用域或复合语句块。

### Lines 1905-1932

````tablegen

// V68 HVX Instructions.

let Features = HVXV68.Features in {
  def V6_v6mpyhubs10 : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, unsigned _Constant int)">;
  def V6_v6mpyhubs10_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<64, int>, unsigned _Constant int)">;
  def V6_v6mpyhubs10_vxx : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, _Vector<32, int>, unsigned _Constant int)">;
  def V6_v6mpyhubs10_vxx_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<64, int>, _Vector<64, int>, unsigned _Constant int)">;
  def V6_v6mpyvubs10 : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, unsigned _Constant int)">;
  def V6_v6mpyvubs10_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<64, int>, unsigned _Constant int)">;
  def V6_v6mpyvubs10_vxx : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, _Vector<32, int>, unsigned _Constant int)">;
  def V6_v6mpyvubs10_vxx_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<64, int>, _Vector<64, int>, unsigned _Constant int)">;
  def V6_vabs_hf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vabs_hf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vabs_sf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vabs_sf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vadd_hf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vadd_hf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vadd_hf_hf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vadd_hf_hf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vadd_qf16 : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vadd_qf16_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vadd_qf16_mix : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vadd_qf16_mix_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vadd_qf32 : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vadd_qf32_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vadd_qf32_mix : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vadd_qf32_mix_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
````
- **L1905 EN**: Blank line separating nearby declarations or logic blocks.
  **L1905 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1906 EN**: Comment explains nearby logic, constraints, or intent: `V68 HVX Instructions.`.
  **L1906 CN**: 注释解释附近代码的逻辑、约束或设计意图：`V68 HVX Instructions.`。
- **L1907 EN**: Blank line separating nearby declarations or logic blocks.
  **L1907 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1908 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = HVXV68.Features in {`.
  **L1908 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = HVXV68.Features in {`。
- **L1909 EN**: Declares TableGen def record `V6_v6mpyhubs10`.
  **L1909 CN**: 声明 TableGen def 记录 `V6_v6mpyhubs10`。
- **L1910 EN**: Declares TableGen def record `V6_v6mpyhubs10_128B`.
  **L1910 CN**: 声明 TableGen def 记录 `V6_v6mpyhubs10_128B`。
- **L1911 EN**: Declares TableGen def record `V6_v6mpyhubs10_vxx`.
  **L1911 CN**: 声明 TableGen def 记录 `V6_v6mpyhubs10_vxx`。
- **L1912 EN**: Declares TableGen def record `V6_v6mpyhubs10_vxx_128B`.
  **L1912 CN**: 声明 TableGen def 记录 `V6_v6mpyhubs10_vxx_128B`。
- **L1913 EN**: Declares TableGen def record `V6_v6mpyvubs10`.
  **L1913 CN**: 声明 TableGen def 记录 `V6_v6mpyvubs10`。
- **L1914 EN**: Declares TableGen def record `V6_v6mpyvubs10_128B`.
  **L1914 CN**: 声明 TableGen def 记录 `V6_v6mpyvubs10_128B`。
- **L1915 EN**: Declares TableGen def record `V6_v6mpyvubs10_vxx`.
  **L1915 CN**: 声明 TableGen def 记录 `V6_v6mpyvubs10_vxx`。
- **L1916 EN**: Declares TableGen def record `V6_v6mpyvubs10_vxx_128B`.
  **L1916 CN**: 声明 TableGen def 记录 `V6_v6mpyvubs10_vxx_128B`。
- **L1917 EN**: Declares TableGen def record `V6_vabs_hf`.
  **L1917 CN**: 声明 TableGen def 记录 `V6_vabs_hf`。
- **L1918 EN**: Declares TableGen def record `V6_vabs_hf_128B`.
  **L1918 CN**: 声明 TableGen def 记录 `V6_vabs_hf_128B`。
- **L1919 EN**: Declares TableGen def record `V6_vabs_sf`.
  **L1919 CN**: 声明 TableGen def 记录 `V6_vabs_sf`。
- **L1920 EN**: Declares TableGen def record `V6_vabs_sf_128B`.
  **L1920 CN**: 声明 TableGen def 记录 `V6_vabs_sf_128B`。
- **L1921 EN**: Declares TableGen def record `V6_vadd_hf`.
  **L1921 CN**: 声明 TableGen def 记录 `V6_vadd_hf`。
- **L1922 EN**: Declares TableGen def record `V6_vadd_hf_128B`.
  **L1922 CN**: 声明 TableGen def 记录 `V6_vadd_hf_128B`。
- **L1923 EN**: Declares TableGen def record `V6_vadd_hf_hf`.
  **L1923 CN**: 声明 TableGen def 记录 `V6_vadd_hf_hf`。
- **L1924 EN**: Declares TableGen def record `V6_vadd_hf_hf_128B`.
  **L1924 CN**: 声明 TableGen def 记录 `V6_vadd_hf_hf_128B`。
- **L1925 EN**: Declares TableGen def record `V6_vadd_qf16`.
  **L1925 CN**: 声明 TableGen def 记录 `V6_vadd_qf16`。
- **L1926 EN**: Declares TableGen def record `V6_vadd_qf16_128B`.
  **L1926 CN**: 声明 TableGen def 记录 `V6_vadd_qf16_128B`。
- **L1927 EN**: Declares TableGen def record `V6_vadd_qf16_mix`.
  **L1927 CN**: 声明 TableGen def 记录 `V6_vadd_qf16_mix`。
- **L1928 EN**: Declares TableGen def record `V6_vadd_qf16_mix_128B`.
  **L1928 CN**: 声明 TableGen def 记录 `V6_vadd_qf16_mix_128B`。
- **L1929 EN**: Declares TableGen def record `V6_vadd_qf32`.
  **L1929 CN**: 声明 TableGen def 记录 `V6_vadd_qf32`。
- **L1930 EN**: Declares TableGen def record `V6_vadd_qf32_128B`.
  **L1930 CN**: 声明 TableGen def 记录 `V6_vadd_qf32_128B`。
- **L1931 EN**: Declares TableGen def record `V6_vadd_qf32_mix`.
  **L1931 CN**: 声明 TableGen def 记录 `V6_vadd_qf32_mix`。
- **L1932 EN**: Declares TableGen def record `V6_vadd_qf32_mix_128B`.
  **L1932 CN**: 声明 TableGen def 记录 `V6_vadd_qf32_mix_128B`。

### Lines 1933-1960

````tablegen
  def V6_vadd_sf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vadd_sf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vadd_sf_hf : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vadd_sf_hf_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vadd_sf_sf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vadd_sf_sf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vassign_fp : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vassign_fp_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vconv_hf_qf16 : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vconv_hf_qf16_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vconv_hf_qf32 : HexagonBuiltin<"_Vector<16, int>(_Vector<32, int>)">;
  def V6_vconv_hf_qf32_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<64, int>)">;
  def V6_vconv_sf_qf32 : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vconv_sf_qf32_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vcvt_b_hf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vcvt_b_hf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vcvt_h_hf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vcvt_h_hf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vcvt_hf_b : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>)">;
  def V6_vcvt_hf_b_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>)">;
  def V6_vcvt_hf_h : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vcvt_hf_h_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vcvt_hf_sf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vcvt_hf_sf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vcvt_hf_ub : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>)">;
  def V6_vcvt_hf_ub_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>)">;
  def V6_vcvt_hf_uh : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vcvt_hf_uh_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
````
- **L1933 EN**: Declares TableGen def record `V6_vadd_sf`.
  **L1933 CN**: 声明 TableGen def 记录 `V6_vadd_sf`。
- **L1934 EN**: Declares TableGen def record `V6_vadd_sf_128B`.
  **L1934 CN**: 声明 TableGen def 记录 `V6_vadd_sf_128B`。
- **L1935 EN**: Declares TableGen def record `V6_vadd_sf_hf`.
  **L1935 CN**: 声明 TableGen def 记录 `V6_vadd_sf_hf`。
- **L1936 EN**: Declares TableGen def record `V6_vadd_sf_hf_128B`.
  **L1936 CN**: 声明 TableGen def 记录 `V6_vadd_sf_hf_128B`。
- **L1937 EN**: Declares TableGen def record `V6_vadd_sf_sf`.
  **L1937 CN**: 声明 TableGen def 记录 `V6_vadd_sf_sf`。
- **L1938 EN**: Declares TableGen def record `V6_vadd_sf_sf_128B`.
  **L1938 CN**: 声明 TableGen def 记录 `V6_vadd_sf_sf_128B`。
- **L1939 EN**: Declares TableGen def record `V6_vassign_fp`.
  **L1939 CN**: 声明 TableGen def 记录 `V6_vassign_fp`。
- **L1940 EN**: Declares TableGen def record `V6_vassign_fp_128B`.
  **L1940 CN**: 声明 TableGen def 记录 `V6_vassign_fp_128B`。
- **L1941 EN**: Declares TableGen def record `V6_vconv_hf_qf16`.
  **L1941 CN**: 声明 TableGen def 记录 `V6_vconv_hf_qf16`。
- **L1942 EN**: Declares TableGen def record `V6_vconv_hf_qf16_128B`.
  **L1942 CN**: 声明 TableGen def 记录 `V6_vconv_hf_qf16_128B`。
- **L1943 EN**: Declares TableGen def record `V6_vconv_hf_qf32`.
  **L1943 CN**: 声明 TableGen def 记录 `V6_vconv_hf_qf32`。
- **L1944 EN**: Declares TableGen def record `V6_vconv_hf_qf32_128B`.
  **L1944 CN**: 声明 TableGen def 记录 `V6_vconv_hf_qf32_128B`。
- **L1945 EN**: Declares TableGen def record `V6_vconv_sf_qf32`.
  **L1945 CN**: 声明 TableGen def 记录 `V6_vconv_sf_qf32`。
- **L1946 EN**: Declares TableGen def record `V6_vconv_sf_qf32_128B`.
  **L1946 CN**: 声明 TableGen def 记录 `V6_vconv_sf_qf32_128B`。
- **L1947 EN**: Declares TableGen def record `V6_vcvt_b_hf`.
  **L1947 CN**: 声明 TableGen def 记录 `V6_vcvt_b_hf`。
- **L1948 EN**: Declares TableGen def record `V6_vcvt_b_hf_128B`.
  **L1948 CN**: 声明 TableGen def 记录 `V6_vcvt_b_hf_128B`。
- **L1949 EN**: Declares TableGen def record `V6_vcvt_h_hf`.
  **L1949 CN**: 声明 TableGen def 记录 `V6_vcvt_h_hf`。
- **L1950 EN**: Declares TableGen def record `V6_vcvt_h_hf_128B`.
  **L1950 CN**: 声明 TableGen def 记录 `V6_vcvt_h_hf_128B`。
- **L1951 EN**: Declares TableGen def record `V6_vcvt_hf_b`.
  **L1951 CN**: 声明 TableGen def 记录 `V6_vcvt_hf_b`。
- **L1952 EN**: Declares TableGen def record `V6_vcvt_hf_b_128B`.
  **L1952 CN**: 声明 TableGen def 记录 `V6_vcvt_hf_b_128B`。
- **L1953 EN**: Declares TableGen def record `V6_vcvt_hf_h`.
  **L1953 CN**: 声明 TableGen def 记录 `V6_vcvt_hf_h`。
- **L1954 EN**: Declares TableGen def record `V6_vcvt_hf_h_128B`.
  **L1954 CN**: 声明 TableGen def 记录 `V6_vcvt_hf_h_128B`。
- **L1955 EN**: Declares TableGen def record `V6_vcvt_hf_sf`.
  **L1955 CN**: 声明 TableGen def 记录 `V6_vcvt_hf_sf`。
- **L1956 EN**: Declares TableGen def record `V6_vcvt_hf_sf_128B`.
  **L1956 CN**: 声明 TableGen def 记录 `V6_vcvt_hf_sf_128B`。
- **L1957 EN**: Declares TableGen def record `V6_vcvt_hf_ub`.
  **L1957 CN**: 声明 TableGen def 记录 `V6_vcvt_hf_ub`。
- **L1958 EN**: Declares TableGen def record `V6_vcvt_hf_ub_128B`.
  **L1958 CN**: 声明 TableGen def 记录 `V6_vcvt_hf_ub_128B`。
- **L1959 EN**: Declares TableGen def record `V6_vcvt_hf_uh`.
  **L1959 CN**: 声明 TableGen def 记录 `V6_vcvt_hf_uh`。
- **L1960 EN**: Declares TableGen def record `V6_vcvt_hf_uh_128B`.
  **L1960 CN**: 声明 TableGen def 记录 `V6_vcvt_hf_uh_128B`。

### Lines 1961-1988

````tablegen
  def V6_vcvt_sf_hf : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>)">;
  def V6_vcvt_sf_hf_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>)">;
  def V6_vcvt_ub_hf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vcvt_ub_hf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vcvt_uh_hf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vcvt_uh_hf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vdmpy_sf_hf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vdmpy_sf_hf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vdmpy_sf_hf_acc : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vdmpy_sf_hf_acc_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vfmax_hf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vfmax_hf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vfmax_sf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vfmax_sf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vfmin_hf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vfmin_hf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vfmin_sf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vfmin_sf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vfneg_hf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vfneg_hf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vfneg_sf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vfneg_sf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vgthf : HexagonBuiltin<"_Vector<64, bool>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vgthf_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vgthf_and : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vgthf_and_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vgthf_or : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vgthf_or_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
````
- **L1961 EN**: Declares TableGen def record `V6_vcvt_sf_hf`.
  **L1961 CN**: 声明 TableGen def 记录 `V6_vcvt_sf_hf`。
- **L1962 EN**: Declares TableGen def record `V6_vcvt_sf_hf_128B`.
  **L1962 CN**: 声明 TableGen def 记录 `V6_vcvt_sf_hf_128B`。
- **L1963 EN**: Declares TableGen def record `V6_vcvt_ub_hf`.
  **L1963 CN**: 声明 TableGen def 记录 `V6_vcvt_ub_hf`。
- **L1964 EN**: Declares TableGen def record `V6_vcvt_ub_hf_128B`.
  **L1964 CN**: 声明 TableGen def 记录 `V6_vcvt_ub_hf_128B`。
- **L1965 EN**: Declares TableGen def record `V6_vcvt_uh_hf`.
  **L1965 CN**: 声明 TableGen def 记录 `V6_vcvt_uh_hf`。
- **L1966 EN**: Declares TableGen def record `V6_vcvt_uh_hf_128B`.
  **L1966 CN**: 声明 TableGen def 记录 `V6_vcvt_uh_hf_128B`。
- **L1967 EN**: Declares TableGen def record `V6_vdmpy_sf_hf`.
  **L1967 CN**: 声明 TableGen def 记录 `V6_vdmpy_sf_hf`。
- **L1968 EN**: Declares TableGen def record `V6_vdmpy_sf_hf_128B`.
  **L1968 CN**: 声明 TableGen def 记录 `V6_vdmpy_sf_hf_128B`。
- **L1969 EN**: Declares TableGen def record `V6_vdmpy_sf_hf_acc`.
  **L1969 CN**: 声明 TableGen def 记录 `V6_vdmpy_sf_hf_acc`。
- **L1970 EN**: Declares TableGen def record `V6_vdmpy_sf_hf_acc_128B`.
  **L1970 CN**: 声明 TableGen def 记录 `V6_vdmpy_sf_hf_acc_128B`。
- **L1971 EN**: Declares TableGen def record `V6_vfmax_hf`.
  **L1971 CN**: 声明 TableGen def 记录 `V6_vfmax_hf`。
- **L1972 EN**: Declares TableGen def record `V6_vfmax_hf_128B`.
  **L1972 CN**: 声明 TableGen def 记录 `V6_vfmax_hf_128B`。
- **L1973 EN**: Declares TableGen def record `V6_vfmax_sf`.
  **L1973 CN**: 声明 TableGen def 记录 `V6_vfmax_sf`。
- **L1974 EN**: Declares TableGen def record `V6_vfmax_sf_128B`.
  **L1974 CN**: 声明 TableGen def 记录 `V6_vfmax_sf_128B`。
- **L1975 EN**: Declares TableGen def record `V6_vfmin_hf`.
  **L1975 CN**: 声明 TableGen def 记录 `V6_vfmin_hf`。
- **L1976 EN**: Declares TableGen def record `V6_vfmin_hf_128B`.
  **L1976 CN**: 声明 TableGen def 记录 `V6_vfmin_hf_128B`。
- **L1977 EN**: Declares TableGen def record `V6_vfmin_sf`.
  **L1977 CN**: 声明 TableGen def 记录 `V6_vfmin_sf`。
- **L1978 EN**: Declares TableGen def record `V6_vfmin_sf_128B`.
  **L1978 CN**: 声明 TableGen def 记录 `V6_vfmin_sf_128B`。
- **L1979 EN**: Declares TableGen def record `V6_vfneg_hf`.
  **L1979 CN**: 声明 TableGen def 记录 `V6_vfneg_hf`。
- **L1980 EN**: Declares TableGen def record `V6_vfneg_hf_128B`.
  **L1980 CN**: 声明 TableGen def 记录 `V6_vfneg_hf_128B`。
- **L1981 EN**: Declares TableGen def record `V6_vfneg_sf`.
  **L1981 CN**: 声明 TableGen def 记录 `V6_vfneg_sf`。
- **L1982 EN**: Declares TableGen def record `V6_vfneg_sf_128B`.
  **L1982 CN**: 声明 TableGen def 记录 `V6_vfneg_sf_128B`。
- **L1983 EN**: Declares TableGen def record `V6_vgthf`.
  **L1983 CN**: 声明 TableGen def 记录 `V6_vgthf`。
- **L1984 EN**: Declares TableGen def record `V6_vgthf_128B`.
  **L1984 CN**: 声明 TableGen def 记录 `V6_vgthf_128B`。
- **L1985 EN**: Declares TableGen def record `V6_vgthf_and`.
  **L1985 CN**: 声明 TableGen def 记录 `V6_vgthf_and`。
- **L1986 EN**: Declares TableGen def record `V6_vgthf_and_128B`.
  **L1986 CN**: 声明 TableGen def 记录 `V6_vgthf_and_128B`。
- **L1987 EN**: Declares TableGen def record `V6_vgthf_or`.
  **L1987 CN**: 声明 TableGen def 记录 `V6_vgthf_or`。
- **L1988 EN**: Declares TableGen def record `V6_vgthf_or_128B`.
  **L1988 CN**: 声明 TableGen def 记录 `V6_vgthf_or_128B`。

### Lines 1989-2016

````tablegen
  def V6_vgthf_xor : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vgthf_xor_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vgtsf : HexagonBuiltin<"_Vector<64, bool>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vgtsf_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vgtsf_and : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vgtsf_and_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vgtsf_or : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vgtsf_or_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vgtsf_xor : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vgtsf_xor_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vmax_hf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vmax_hf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vmax_sf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vmax_sf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vmin_hf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vmin_hf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vmin_sf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vmin_sf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpy_hf_hf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpy_hf_hf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpy_hf_hf_acc : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpy_hf_hf_acc_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpy_qf16 : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpy_qf16_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpy_qf16_hf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpy_qf16_hf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpy_qf16_mix_hf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpy_qf16_mix_hf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
````
- **L1989 EN**: Declares TableGen def record `V6_vgthf_xor`.
  **L1989 CN**: 声明 TableGen def 记录 `V6_vgthf_xor`。
- **L1990 EN**: Declares TableGen def record `V6_vgthf_xor_128B`.
  **L1990 CN**: 声明 TableGen def 记录 `V6_vgthf_xor_128B`。
- **L1991 EN**: Declares TableGen def record `V6_vgtsf`.
  **L1991 CN**: 声明 TableGen def 记录 `V6_vgtsf`。
- **L1992 EN**: Declares TableGen def record `V6_vgtsf_128B`.
  **L1992 CN**: 声明 TableGen def 记录 `V6_vgtsf_128B`。
- **L1993 EN**: Declares TableGen def record `V6_vgtsf_and`.
  **L1993 CN**: 声明 TableGen def 记录 `V6_vgtsf_and`。
- **L1994 EN**: Declares TableGen def record `V6_vgtsf_and_128B`.
  **L1994 CN**: 声明 TableGen def 记录 `V6_vgtsf_and_128B`。
- **L1995 EN**: Declares TableGen def record `V6_vgtsf_or`.
  **L1995 CN**: 声明 TableGen def 记录 `V6_vgtsf_or`。
- **L1996 EN**: Declares TableGen def record `V6_vgtsf_or_128B`.
  **L1996 CN**: 声明 TableGen def 记录 `V6_vgtsf_or_128B`。
- **L1997 EN**: Declares TableGen def record `V6_vgtsf_xor`.
  **L1997 CN**: 声明 TableGen def 记录 `V6_vgtsf_xor`。
- **L1998 EN**: Declares TableGen def record `V6_vgtsf_xor_128B`.
  **L1998 CN**: 声明 TableGen def 记录 `V6_vgtsf_xor_128B`。
- **L1999 EN**: Declares TableGen def record `V6_vmax_hf`.
  **L1999 CN**: 声明 TableGen def 记录 `V6_vmax_hf`。
- **L2000 EN**: Declares TableGen def record `V6_vmax_hf_128B`.
  **L2000 CN**: 声明 TableGen def 记录 `V6_vmax_hf_128B`。
- **L2001 EN**: Declares TableGen def record `V6_vmax_sf`.
  **L2001 CN**: 声明 TableGen def 记录 `V6_vmax_sf`。
- **L2002 EN**: Declares TableGen def record `V6_vmax_sf_128B`.
  **L2002 CN**: 声明 TableGen def 记录 `V6_vmax_sf_128B`。
- **L2003 EN**: Declares TableGen def record `V6_vmin_hf`.
  **L2003 CN**: 声明 TableGen def 记录 `V6_vmin_hf`。
- **L2004 EN**: Declares TableGen def record `V6_vmin_hf_128B`.
  **L2004 CN**: 声明 TableGen def 记录 `V6_vmin_hf_128B`。
- **L2005 EN**: Declares TableGen def record `V6_vmin_sf`.
  **L2005 CN**: 声明 TableGen def 记录 `V6_vmin_sf`。
- **L2006 EN**: Declares TableGen def record `V6_vmin_sf_128B`.
  **L2006 CN**: 声明 TableGen def 记录 `V6_vmin_sf_128B`。
- **L2007 EN**: Declares TableGen def record `V6_vmpy_hf_hf`.
  **L2007 CN**: 声明 TableGen def 记录 `V6_vmpy_hf_hf`。
- **L2008 EN**: Declares TableGen def record `V6_vmpy_hf_hf_128B`.
  **L2008 CN**: 声明 TableGen def 记录 `V6_vmpy_hf_hf_128B`。
- **L2009 EN**: Declares TableGen def record `V6_vmpy_hf_hf_acc`.
  **L2009 CN**: 声明 TableGen def 记录 `V6_vmpy_hf_hf_acc`。
- **L2010 EN**: Declares TableGen def record `V6_vmpy_hf_hf_acc_128B`.
  **L2010 CN**: 声明 TableGen def 记录 `V6_vmpy_hf_hf_acc_128B`。
- **L2011 EN**: Declares TableGen def record `V6_vmpy_qf16`.
  **L2011 CN**: 声明 TableGen def 记录 `V6_vmpy_qf16`。
- **L2012 EN**: Declares TableGen def record `V6_vmpy_qf16_128B`.
  **L2012 CN**: 声明 TableGen def 记录 `V6_vmpy_qf16_128B`。
- **L2013 EN**: Declares TableGen def record `V6_vmpy_qf16_hf`.
  **L2013 CN**: 声明 TableGen def 记录 `V6_vmpy_qf16_hf`。
- **L2014 EN**: Declares TableGen def record `V6_vmpy_qf16_hf_128B`.
  **L2014 CN**: 声明 TableGen def 记录 `V6_vmpy_qf16_hf_128B`。
- **L2015 EN**: Declares TableGen def record `V6_vmpy_qf16_mix_hf`.
  **L2015 CN**: 声明 TableGen def 记录 `V6_vmpy_qf16_mix_hf`。
- **L2016 EN**: Declares TableGen def record `V6_vmpy_qf16_mix_hf_128B`.
  **L2016 CN**: 声明 TableGen def 记录 `V6_vmpy_qf16_mix_hf_128B`。

### Lines 2017-2044

````tablegen
  def V6_vmpy_qf32 : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpy_qf32_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpy_qf32_hf : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpy_qf32_hf_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpy_qf32_mix_hf : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpy_qf32_mix_hf_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpy_qf32_qf16 : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpy_qf32_qf16_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpy_qf32_sf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpy_qf32_sf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpy_sf_hf : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpy_sf_hf_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpy_sf_hf_acc : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpy_sf_hf_acc_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpy_sf_sf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpy_sf_sf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vsub_hf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vsub_hf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vsub_hf_hf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vsub_hf_hf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vsub_qf16 : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vsub_qf16_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vsub_qf16_mix : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vsub_qf16_mix_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vsub_qf32 : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vsub_qf32_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vsub_qf32_mix : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vsub_qf32_mix_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
````
- **L2017 EN**: Declares TableGen def record `V6_vmpy_qf32`.
  **L2017 CN**: 声明 TableGen def 记录 `V6_vmpy_qf32`。
- **L2018 EN**: Declares TableGen def record `V6_vmpy_qf32_128B`.
  **L2018 CN**: 声明 TableGen def 记录 `V6_vmpy_qf32_128B`。
- **L2019 EN**: Declares TableGen def record `V6_vmpy_qf32_hf`.
  **L2019 CN**: 声明 TableGen def 记录 `V6_vmpy_qf32_hf`。
- **L2020 EN**: Declares TableGen def record `V6_vmpy_qf32_hf_128B`.
  **L2020 CN**: 声明 TableGen def 记录 `V6_vmpy_qf32_hf_128B`。
- **L2021 EN**: Declares TableGen def record `V6_vmpy_qf32_mix_hf`.
  **L2021 CN**: 声明 TableGen def 记录 `V6_vmpy_qf32_mix_hf`。
- **L2022 EN**: Declares TableGen def record `V6_vmpy_qf32_mix_hf_128B`.
  **L2022 CN**: 声明 TableGen def 记录 `V6_vmpy_qf32_mix_hf_128B`。
- **L2023 EN**: Declares TableGen def record `V6_vmpy_qf32_qf16`.
  **L2023 CN**: 声明 TableGen def 记录 `V6_vmpy_qf32_qf16`。
- **L2024 EN**: Declares TableGen def record `V6_vmpy_qf32_qf16_128B`.
  **L2024 CN**: 声明 TableGen def 记录 `V6_vmpy_qf32_qf16_128B`。
- **L2025 EN**: Declares TableGen def record `V6_vmpy_qf32_sf`.
  **L2025 CN**: 声明 TableGen def 记录 `V6_vmpy_qf32_sf`。
- **L2026 EN**: Declares TableGen def record `V6_vmpy_qf32_sf_128B`.
  **L2026 CN**: 声明 TableGen def 记录 `V6_vmpy_qf32_sf_128B`。
- **L2027 EN**: Declares TableGen def record `V6_vmpy_sf_hf`.
  **L2027 CN**: 声明 TableGen def 记录 `V6_vmpy_sf_hf`。
- **L2028 EN**: Declares TableGen def record `V6_vmpy_sf_hf_128B`.
  **L2028 CN**: 声明 TableGen def 记录 `V6_vmpy_sf_hf_128B`。
- **L2029 EN**: Declares TableGen def record `V6_vmpy_sf_hf_acc`.
  **L2029 CN**: 声明 TableGen def 记录 `V6_vmpy_sf_hf_acc`。
- **L2030 EN**: Declares TableGen def record `V6_vmpy_sf_hf_acc_128B`.
  **L2030 CN**: 声明 TableGen def 记录 `V6_vmpy_sf_hf_acc_128B`。
- **L2031 EN**: Declares TableGen def record `V6_vmpy_sf_sf`.
  **L2031 CN**: 声明 TableGen def 记录 `V6_vmpy_sf_sf`。
- **L2032 EN**: Declares TableGen def record `V6_vmpy_sf_sf_128B`.
  **L2032 CN**: 声明 TableGen def 记录 `V6_vmpy_sf_sf_128B`。
- **L2033 EN**: Declares TableGen def record `V6_vsub_hf`.
  **L2033 CN**: 声明 TableGen def 记录 `V6_vsub_hf`。
- **L2034 EN**: Declares TableGen def record `V6_vsub_hf_128B`.
  **L2034 CN**: 声明 TableGen def 记录 `V6_vsub_hf_128B`。
- **L2035 EN**: Declares TableGen def record `V6_vsub_hf_hf`.
  **L2035 CN**: 声明 TableGen def 记录 `V6_vsub_hf_hf`。
- **L2036 EN**: Declares TableGen def record `V6_vsub_hf_hf_128B`.
  **L2036 CN**: 声明 TableGen def 记录 `V6_vsub_hf_hf_128B`。
- **L2037 EN**: Declares TableGen def record `V6_vsub_qf16`.
  **L2037 CN**: 声明 TableGen def 记录 `V6_vsub_qf16`。
- **L2038 EN**: Declares TableGen def record `V6_vsub_qf16_128B`.
  **L2038 CN**: 声明 TableGen def 记录 `V6_vsub_qf16_128B`。
- **L2039 EN**: Declares TableGen def record `V6_vsub_qf16_mix`.
  **L2039 CN**: 声明 TableGen def 记录 `V6_vsub_qf16_mix`。
- **L2040 EN**: Declares TableGen def record `V6_vsub_qf16_mix_128B`.
  **L2040 CN**: 声明 TableGen def 记录 `V6_vsub_qf16_mix_128B`。
- **L2041 EN**: Declares TableGen def record `V6_vsub_qf32`.
  **L2041 CN**: 声明 TableGen def 记录 `V6_vsub_qf32`。
- **L2042 EN**: Declares TableGen def record `V6_vsub_qf32_128B`.
  **L2042 CN**: 声明 TableGen def 记录 `V6_vsub_qf32_128B`。
- **L2043 EN**: Declares TableGen def record `V6_vsub_qf32_mix`.
  **L2043 CN**: 声明 TableGen def 记录 `V6_vsub_qf32_mix`。
- **L2044 EN**: Declares TableGen def record `V6_vsub_qf32_mix_128B`.
  **L2044 CN**: 声明 TableGen def 记录 `V6_vsub_qf32_mix_128B`。

### Lines 2045-2072

````tablegen
  def V6_vsub_sf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vsub_sf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vsub_sf_hf : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vsub_sf_hf_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vsub_sf_sf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vsub_sf_sf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
}

// V69 HVX Instructions.

let Features = HVXV69.Features in {
  def V6_vasrvuhubrndsat : HexagonBuiltin<"_Vector<16, int>(_Vector<32, int>, _Vector<16, int>)">;
  def V6_vasrvuhubrndsat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<64, int>, _Vector<32, int>)">;
  def V6_vasrvuhubsat : HexagonBuiltin<"_Vector<16, int>(_Vector<32, int>, _Vector<16, int>)">;
  def V6_vasrvuhubsat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<64, int>, _Vector<32, int>)">;
  def V6_vasrvwuhrndsat : HexagonBuiltin<"_Vector<16, int>(_Vector<32, int>, _Vector<16, int>)">;
  def V6_vasrvwuhrndsat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<64, int>, _Vector<32, int>)">;
  def V6_vasrvwuhsat : HexagonBuiltin<"_Vector<16, int>(_Vector<32, int>, _Vector<16, int>)">;
  def V6_vasrvwuhsat_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<64, int>, _Vector<32, int>)">;
  def V6_vmpyuhvs : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpyuhvs_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
}

// V73 HVX Instructions.

let Features = HVXV73.Features in {
  def V6_vadd_sf_bf : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vadd_sf_bf_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>, _Vector<32, int>)">;
````
- **L2045 EN**: Declares TableGen def record `V6_vsub_sf`.
  **L2045 CN**: 声明 TableGen def 记录 `V6_vsub_sf`。
- **L2046 EN**: Declares TableGen def record `V6_vsub_sf_128B`.
  **L2046 CN**: 声明 TableGen def 记录 `V6_vsub_sf_128B`。
- **L2047 EN**: Declares TableGen def record `V6_vsub_sf_hf`.
  **L2047 CN**: 声明 TableGen def 记录 `V6_vsub_sf_hf`。
- **L2048 EN**: Declares TableGen def record `V6_vsub_sf_hf_128B`.
  **L2048 CN**: 声明 TableGen def 记录 `V6_vsub_sf_hf_128B`。
- **L2049 EN**: Declares TableGen def record `V6_vsub_sf_sf`.
  **L2049 CN**: 声明 TableGen def 记录 `V6_vsub_sf_sf`。
- **L2050 EN**: Declares TableGen def record `V6_vsub_sf_sf_128B`.
  **L2050 CN**: 声明 TableGen def 记录 `V6_vsub_sf_sf_128B`。
- **L2051 EN**: Closes the current lexical scope or compound statement.
  **L2051 CN**: 结束当前词法作用域或复合语句块。
- **L2052 EN**: Blank line separating nearby declarations or logic blocks.
  **L2052 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2053 EN**: Comment explains nearby logic, constraints, or intent: `V69 HVX Instructions.`.
  **L2053 CN**: 注释解释附近代码的逻辑、约束或设计意图：`V69 HVX Instructions.`。
- **L2054 EN**: Blank line separating nearby declarations or logic blocks.
  **L2054 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2055 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = HVXV69.Features in {`.
  **L2055 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = HVXV69.Features in {`。
- **L2056 EN**: Declares TableGen def record `V6_vasrvuhubrndsat`.
  **L2056 CN**: 声明 TableGen def 记录 `V6_vasrvuhubrndsat`。
- **L2057 EN**: Declares TableGen def record `V6_vasrvuhubrndsat_128B`.
  **L2057 CN**: 声明 TableGen def 记录 `V6_vasrvuhubrndsat_128B`。
- **L2058 EN**: Declares TableGen def record `V6_vasrvuhubsat`.
  **L2058 CN**: 声明 TableGen def 记录 `V6_vasrvuhubsat`。
- **L2059 EN**: Declares TableGen def record `V6_vasrvuhubsat_128B`.
  **L2059 CN**: 声明 TableGen def 记录 `V6_vasrvuhubsat_128B`。
- **L2060 EN**: Declares TableGen def record `V6_vasrvwuhrndsat`.
  **L2060 CN**: 声明 TableGen def 记录 `V6_vasrvwuhrndsat`。
- **L2061 EN**: Declares TableGen def record `V6_vasrvwuhrndsat_128B`.
  **L2061 CN**: 声明 TableGen def 记录 `V6_vasrvwuhrndsat_128B`。
- **L2062 EN**: Declares TableGen def record `V6_vasrvwuhsat`.
  **L2062 CN**: 声明 TableGen def 记录 `V6_vasrvwuhsat`。
- **L2063 EN**: Declares TableGen def record `V6_vasrvwuhsat_128B`.
  **L2063 CN**: 声明 TableGen def 记录 `V6_vasrvwuhsat_128B`。
- **L2064 EN**: Declares TableGen def record `V6_vmpyuhvs`.
  **L2064 CN**: 声明 TableGen def 记录 `V6_vmpyuhvs`。
- **L2065 EN**: Declares TableGen def record `V6_vmpyuhvs_128B`.
  **L2065 CN**: 声明 TableGen def 记录 `V6_vmpyuhvs_128B`。
- **L2066 EN**: Closes the current lexical scope or compound statement.
  **L2066 CN**: 结束当前词法作用域或复合语句块。
- **L2067 EN**: Blank line separating nearby declarations or logic blocks.
  **L2067 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2068 EN**: Comment explains nearby logic, constraints, or intent: `V73 HVX Instructions.`.
  **L2068 CN**: 注释解释附近代码的逻辑、约束或设计意图：`V73 HVX Instructions.`。
- **L2069 EN**: Blank line separating nearby declarations or logic blocks.
  **L2069 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2070 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = HVXV73.Features in {`.
  **L2070 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = HVXV73.Features in {`。
- **L2071 EN**: Declares TableGen def record `V6_vadd_sf_bf`.
  **L2071 CN**: 声明 TableGen def 记录 `V6_vadd_sf_bf`。
- **L2072 EN**: Declares TableGen def record `V6_vadd_sf_bf_128B`.
  **L2072 CN**: 声明 TableGen def 记录 `V6_vadd_sf_bf_128B`。

### Lines 2073-2100

````tablegen
  def V6_vconv_h_hf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vconv_h_hf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vconv_hf_h : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vconv_hf_h_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vconv_sf_w : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vconv_sf_w_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vconv_w_sf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vconv_w_sf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vcvt_bf_sf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vcvt_bf_sf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vgtbf : HexagonBuiltin<"_Vector<64, bool>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vgtbf_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vgtbf_and : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vgtbf_and_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vgtbf_or : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vgtbf_or_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vgtbf_xor : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vgtbf_xor_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vmax_bf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vmax_bf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vmin_bf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vmin_bf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpy_sf_bf : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpy_sf_bf_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpy_sf_bf_acc : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpy_sf_bf_acc_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vsub_sf_bf : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vsub_sf_bf_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>, _Vector<32, int>)">;
````
- **L2073 EN**: Declares TableGen def record `V6_vconv_h_hf`.
  **L2073 CN**: 声明 TableGen def 记录 `V6_vconv_h_hf`。
- **L2074 EN**: Declares TableGen def record `V6_vconv_h_hf_128B`.
  **L2074 CN**: 声明 TableGen def 记录 `V6_vconv_h_hf_128B`。
- **L2075 EN**: Declares TableGen def record `V6_vconv_hf_h`.
  **L2075 CN**: 声明 TableGen def 记录 `V6_vconv_hf_h`。
- **L2076 EN**: Declares TableGen def record `V6_vconv_hf_h_128B`.
  **L2076 CN**: 声明 TableGen def 记录 `V6_vconv_hf_h_128B`。
- **L2077 EN**: Declares TableGen def record `V6_vconv_sf_w`.
  **L2077 CN**: 声明 TableGen def 记录 `V6_vconv_sf_w`。
- **L2078 EN**: Declares TableGen def record `V6_vconv_sf_w_128B`.
  **L2078 CN**: 声明 TableGen def 记录 `V6_vconv_sf_w_128B`。
- **L2079 EN**: Declares TableGen def record `V6_vconv_w_sf`.
  **L2079 CN**: 声明 TableGen def 记录 `V6_vconv_w_sf`。
- **L2080 EN**: Declares TableGen def record `V6_vconv_w_sf_128B`.
  **L2080 CN**: 声明 TableGen def 记录 `V6_vconv_w_sf_128B`。
- **L2081 EN**: Declares TableGen def record `V6_vcvt_bf_sf`.
  **L2081 CN**: 声明 TableGen def 记录 `V6_vcvt_bf_sf`。
- **L2082 EN**: Declares TableGen def record `V6_vcvt_bf_sf_128B`.
  **L2082 CN**: 声明 TableGen def 记录 `V6_vcvt_bf_sf_128B`。
- **L2083 EN**: Declares TableGen def record `V6_vgtbf`.
  **L2083 CN**: 声明 TableGen def 记录 `V6_vgtbf`。
- **L2084 EN**: Declares TableGen def record `V6_vgtbf_128B`.
  **L2084 CN**: 声明 TableGen def 记录 `V6_vgtbf_128B`。
- **L2085 EN**: Declares TableGen def record `V6_vgtbf_and`.
  **L2085 CN**: 声明 TableGen def 记录 `V6_vgtbf_and`。
- **L2086 EN**: Declares TableGen def record `V6_vgtbf_and_128B`.
  **L2086 CN**: 声明 TableGen def 记录 `V6_vgtbf_and_128B`。
- **L2087 EN**: Declares TableGen def record `V6_vgtbf_or`.
  **L2087 CN**: 声明 TableGen def 记录 `V6_vgtbf_or`。
- **L2088 EN**: Declares TableGen def record `V6_vgtbf_or_128B`.
  **L2088 CN**: 声明 TableGen def 记录 `V6_vgtbf_or_128B`。
- **L2089 EN**: Declares TableGen def record `V6_vgtbf_xor`.
  **L2089 CN**: 声明 TableGen def 记录 `V6_vgtbf_xor`。
- **L2090 EN**: Declares TableGen def record `V6_vgtbf_xor_128B`.
  **L2090 CN**: 声明 TableGen def 记录 `V6_vgtbf_xor_128B`。
- **L2091 EN**: Declares TableGen def record `V6_vmax_bf`.
  **L2091 CN**: 声明 TableGen def 记录 `V6_vmax_bf`。
- **L2092 EN**: Declares TableGen def record `V6_vmax_bf_128B`.
  **L2092 CN**: 声明 TableGen def 记录 `V6_vmax_bf_128B`。
- **L2093 EN**: Declares TableGen def record `V6_vmin_bf`.
  **L2093 CN**: 声明 TableGen def 记录 `V6_vmin_bf`。
- **L2094 EN**: Declares TableGen def record `V6_vmin_bf_128B`.
  **L2094 CN**: 声明 TableGen def 记录 `V6_vmin_bf_128B`。
- **L2095 EN**: Declares TableGen def record `V6_vmpy_sf_bf`.
  **L2095 CN**: 声明 TableGen def 记录 `V6_vmpy_sf_bf`。
- **L2096 EN**: Declares TableGen def record `V6_vmpy_sf_bf_128B`.
  **L2096 CN**: 声明 TableGen def 记录 `V6_vmpy_sf_bf_128B`。
- **L2097 EN**: Declares TableGen def record `V6_vmpy_sf_bf_acc`.
  **L2097 CN**: 声明 TableGen def 记录 `V6_vmpy_sf_bf_acc`。
- **L2098 EN**: Declares TableGen def record `V6_vmpy_sf_bf_acc_128B`.
  **L2098 CN**: 声明 TableGen def 记录 `V6_vmpy_sf_bf_acc_128B`。
- **L2099 EN**: Declares TableGen def record `V6_vsub_sf_bf`.
  **L2099 CN**: 声明 TableGen def 记录 `V6_vsub_sf_bf`。
- **L2100 EN**: Declares TableGen def record `V6_vsub_sf_bf_128B`.
  **L2100 CN**: 声明 TableGen def 记录 `V6_vsub_sf_bf_128B`。

### Lines 2101-2128

````tablegen
}

// V79 HVX Instructions.

let Features = HVXV79.Features in {
  def V6_get_qfext : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, int)">;
  def V6_get_qfext_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, int)">;
  def V6_get_qfext_oracc : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, int)">;
  def V6_get_qfext_oracc_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_set_qfext : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, int)">;
  def V6_set_qfext_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, int)">;
  def V6_vabs_f8 : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vabs_f8_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vadd_hf_f8 : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vadd_hf_f8_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vcvt2_b_hf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vcvt2_b_hf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vcvt2_hf_b : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>)">;
  def V6_vcvt2_hf_b_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>)">;
  def V6_vcvt2_hf_ub : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>)">;
  def V6_vcvt2_hf_ub_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>)">;
  def V6_vcvt2_ub_hf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vcvt2_ub_hf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vcvt_f8_hf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vcvt_f8_hf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vcvt_hf_f8 : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>)">;
  def V6_vcvt_hf_f8_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>)">;
  def V6_vfmax_f8 : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
````
- **L2101 EN**: Closes the current lexical scope or compound statement.
  **L2101 CN**: 结束当前词法作用域或复合语句块。
- **L2102 EN**: Blank line separating nearby declarations or logic blocks.
  **L2102 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2103 EN**: Comment explains nearby logic, constraints, or intent: `V79 HVX Instructions.`.
  **L2103 CN**: 注释解释附近代码的逻辑、约束或设计意图：`V79 HVX Instructions.`。
- **L2104 EN**: Blank line separating nearby declarations or logic blocks.
  **L2104 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2105 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = HVXV79.Features in {`.
  **L2105 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = HVXV79.Features in {`。
- **L2106 EN**: Declares TableGen def record `V6_get_qfext`.
  **L2106 CN**: 声明 TableGen def 记录 `V6_get_qfext`。
- **L2107 EN**: Declares TableGen def record `V6_get_qfext_128B`.
  **L2107 CN**: 声明 TableGen def 记录 `V6_get_qfext_128B`。
- **L2108 EN**: Declares TableGen def record `V6_get_qfext_oracc`.
  **L2108 CN**: 声明 TableGen def 记录 `V6_get_qfext_oracc`。
- **L2109 EN**: Declares TableGen def record `V6_get_qfext_oracc_128B`.
  **L2109 CN**: 声明 TableGen def 记录 `V6_get_qfext_oracc_128B`。
- **L2110 EN**: Declares TableGen def record `V6_set_qfext`.
  **L2110 CN**: 声明 TableGen def 记录 `V6_set_qfext`。
- **L2111 EN**: Declares TableGen def record `V6_set_qfext_128B`.
  **L2111 CN**: 声明 TableGen def 记录 `V6_set_qfext_128B`。
- **L2112 EN**: Declares TableGen def record `V6_vabs_f8`.
  **L2112 CN**: 声明 TableGen def 记录 `V6_vabs_f8`。
- **L2113 EN**: Declares TableGen def record `V6_vabs_f8_128B`.
  **L2113 CN**: 声明 TableGen def 记录 `V6_vabs_f8_128B`。
- **L2114 EN**: Declares TableGen def record `V6_vadd_hf_f8`.
  **L2114 CN**: 声明 TableGen def 记录 `V6_vadd_hf_f8`。
- **L2115 EN**: Declares TableGen def record `V6_vadd_hf_f8_128B`.
  **L2115 CN**: 声明 TableGen def 记录 `V6_vadd_hf_f8_128B`。
- **L2116 EN**: Declares TableGen def record `V6_vcvt2_b_hf`.
  **L2116 CN**: 声明 TableGen def 记录 `V6_vcvt2_b_hf`。
- **L2117 EN**: Declares TableGen def record `V6_vcvt2_b_hf_128B`.
  **L2117 CN**: 声明 TableGen def 记录 `V6_vcvt2_b_hf_128B`。
- **L2118 EN**: Declares TableGen def record `V6_vcvt2_hf_b`.
  **L2118 CN**: 声明 TableGen def 记录 `V6_vcvt2_hf_b`。
- **L2119 EN**: Declares TableGen def record `V6_vcvt2_hf_b_128B`.
  **L2119 CN**: 声明 TableGen def 记录 `V6_vcvt2_hf_b_128B`。
- **L2120 EN**: Declares TableGen def record `V6_vcvt2_hf_ub`.
  **L2120 CN**: 声明 TableGen def 记录 `V6_vcvt2_hf_ub`。
- **L2121 EN**: Declares TableGen def record `V6_vcvt2_hf_ub_128B`.
  **L2121 CN**: 声明 TableGen def 记录 `V6_vcvt2_hf_ub_128B`。
- **L2122 EN**: Declares TableGen def record `V6_vcvt2_ub_hf`.
  **L2122 CN**: 声明 TableGen def 记录 `V6_vcvt2_ub_hf`。
- **L2123 EN**: Declares TableGen def record `V6_vcvt2_ub_hf_128B`.
  **L2123 CN**: 声明 TableGen def 记录 `V6_vcvt2_ub_hf_128B`。
- **L2124 EN**: Declares TableGen def record `V6_vcvt_f8_hf`.
  **L2124 CN**: 声明 TableGen def 记录 `V6_vcvt_f8_hf`。
- **L2125 EN**: Declares TableGen def record `V6_vcvt_f8_hf_128B`.
  **L2125 CN**: 声明 TableGen def 记录 `V6_vcvt_f8_hf_128B`。
- **L2126 EN**: Declares TableGen def record `V6_vcvt_hf_f8`.
  **L2126 CN**: 声明 TableGen def 记录 `V6_vcvt_hf_f8`。
- **L2127 EN**: Declares TableGen def record `V6_vcvt_hf_f8_128B`.
  **L2127 CN**: 声明 TableGen def 记录 `V6_vcvt_hf_f8_128B`。
- **L2128 EN**: Declares TableGen def record `V6_vfmax_f8`.
  **L2128 CN**: 声明 TableGen def 记录 `V6_vfmax_f8`。

### Lines 2129-2156

````tablegen
  def V6_vfmax_f8_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vfmin_f8 : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vfmin_f8_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vfneg_f8 : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vfneg_f8_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vmerge_qf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vmerge_qf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpy_hf_f8 : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpy_hf_f8_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpy_hf_f8_acc : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_vmpy_hf_f8_acc_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<64, int>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vmpy_rt_hf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, int)">;
  def V6_vmpy_rt_hf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, int)">;
  def V6_vmpy_rt_qf16 : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, int)">;
  def V6_vmpy_rt_qf16_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, int)">;
  def V6_vmpy_rt_sf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, int)">;
  def V6_vmpy_rt_sf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, int)">;
  def V6_vsub_hf_f8 : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vsub_hf_f8_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>, _Vector<32, int>)">;
}

// V81 HVX Instructions.
let Features = HVXV81.Features in {
  def V6_vabs_qf16_hf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vabs_qf16_hf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vabs_qf16_qf16 : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vabs_qf16_qf16_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vabs_qf32_qf32 : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
````
- **L2129 EN**: Declares TableGen def record `V6_vfmax_f8_128B`.
  **L2129 CN**: 声明 TableGen def 记录 `V6_vfmax_f8_128B`。
- **L2130 EN**: Declares TableGen def record `V6_vfmin_f8`.
  **L2130 CN**: 声明 TableGen def 记录 `V6_vfmin_f8`。
- **L2131 EN**: Declares TableGen def record `V6_vfmin_f8_128B`.
  **L2131 CN**: 声明 TableGen def 记录 `V6_vfmin_f8_128B`。
- **L2132 EN**: Declares TableGen def record `V6_vfneg_f8`.
  **L2132 CN**: 声明 TableGen def 记录 `V6_vfneg_f8`。
- **L2133 EN**: Declares TableGen def record `V6_vfneg_f8_128B`.
  **L2133 CN**: 声明 TableGen def 记录 `V6_vfneg_f8_128B`。
- **L2134 EN**: Declares TableGen def record `V6_vmerge_qf`.
  **L2134 CN**: 声明 TableGen def 记录 `V6_vmerge_qf`。
- **L2135 EN**: Declares TableGen def record `V6_vmerge_qf_128B`.
  **L2135 CN**: 声明 TableGen def 记录 `V6_vmerge_qf_128B`。
- **L2136 EN**: Declares TableGen def record `V6_vmpy_hf_f8`.
  **L2136 CN**: 声明 TableGen def 记录 `V6_vmpy_hf_f8`。
- **L2137 EN**: Declares TableGen def record `V6_vmpy_hf_f8_128B`.
  **L2137 CN**: 声明 TableGen def 记录 `V6_vmpy_hf_f8_128B`。
- **L2138 EN**: Declares TableGen def record `V6_vmpy_hf_f8_acc`.
  **L2138 CN**: 声明 TableGen def 记录 `V6_vmpy_hf_f8_acc`。
- **L2139 EN**: Declares TableGen def record `V6_vmpy_hf_f8_acc_128B`.
  **L2139 CN**: 声明 TableGen def 记录 `V6_vmpy_hf_f8_acc_128B`。
- **L2140 EN**: Declares TableGen def record `V6_vmpy_rt_hf`.
  **L2140 CN**: 声明 TableGen def 记录 `V6_vmpy_rt_hf`。
- **L2141 EN**: Declares TableGen def record `V6_vmpy_rt_hf_128B`.
  **L2141 CN**: 声明 TableGen def 记录 `V6_vmpy_rt_hf_128B`。
- **L2142 EN**: Declares TableGen def record `V6_vmpy_rt_qf16`.
  **L2142 CN**: 声明 TableGen def 记录 `V6_vmpy_rt_qf16`。
- **L2143 EN**: Declares TableGen def record `V6_vmpy_rt_qf16_128B`.
  **L2143 CN**: 声明 TableGen def 记录 `V6_vmpy_rt_qf16_128B`。
- **L2144 EN**: Declares TableGen def record `V6_vmpy_rt_sf`.
  **L2144 CN**: 声明 TableGen def 记录 `V6_vmpy_rt_sf`。
- **L2145 EN**: Declares TableGen def record `V6_vmpy_rt_sf_128B`.
  **L2145 CN**: 声明 TableGen def 记录 `V6_vmpy_rt_sf_128B`。
- **L2146 EN**: Declares TableGen def record `V6_vsub_hf_f8`.
  **L2146 CN**: 声明 TableGen def 记录 `V6_vsub_hf_f8`。
- **L2147 EN**: Declares TableGen def record `V6_vsub_hf_f8_128B`.
  **L2147 CN**: 声明 TableGen def 记录 `V6_vsub_hf_f8_128B`。
- **L2148 EN**: Closes the current lexical scope or compound statement.
  **L2148 CN**: 结束当前词法作用域或复合语句块。
- **L2149 EN**: Blank line separating nearby declarations or logic blocks.
  **L2149 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2150 EN**: Comment explains nearby logic, constraints, or intent: `V81 HVX Instructions.`.
  **L2150 CN**: 注释解释附近代码的逻辑、约束或设计意图：`V81 HVX Instructions.`。
- **L2151 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = HVXV81.Features in {`.
  **L2151 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = HVXV81.Features in {`。
- **L2152 EN**: Declares TableGen def record `V6_vabs_qf16_hf`.
  **L2152 CN**: 声明 TableGen def 记录 `V6_vabs_qf16_hf`。
- **L2153 EN**: Declares TableGen def record `V6_vabs_qf16_hf_128B`.
  **L2153 CN**: 声明 TableGen def 记录 `V6_vabs_qf16_hf_128B`。
- **L2154 EN**: Declares TableGen def record `V6_vabs_qf16_qf16`.
  **L2154 CN**: 声明 TableGen def 记录 `V6_vabs_qf16_qf16`。
- **L2155 EN**: Declares TableGen def record `V6_vabs_qf16_qf16_128B`.
  **L2155 CN**: 声明 TableGen def 记录 `V6_vabs_qf16_qf16_128B`。
- **L2156 EN**: Declares TableGen def record `V6_vabs_qf32_qf32`.
  **L2156 CN**: 声明 TableGen def 记录 `V6_vabs_qf32_qf32`。

### Lines 2157-2184

````tablegen
  def V6_vabs_qf32_qf32_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vabs_qf32_sf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vabs_qf32_sf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_valign4 : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>, int)">;
  def V6_valign4_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>, int)">;
  def V6_vconv_bf_qf32 : HexagonBuiltin<"_Vector<16, int>(_Vector<32, int>)">;
  def V6_vconv_bf_qf32_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<64, int>)">;
  def V6_vconv_f8_qf16 : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vconv_f8_qf16_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vconv_h_hf_rnd : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vconv_h_hf_rnd_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vconv_qf16_f8 : HexagonBuiltin<"_Vector<32, int>(_Vector<16, int>)">;
  def V6_vconv_qf16_f8_128B : HexagonBuiltin<"_Vector<64, int>(_Vector<32, int>)">;
  def V6_vconv_qf16_hf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vconv_qf16_hf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vconv_qf16_qf16 : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vconv_qf16_qf16_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vconv_qf32_qf32 : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vconv_qf32_qf32_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vconv_qf32_sf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vconv_qf32_sf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_veqhf : HexagonBuiltin<"_Vector<64, bool>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_veqhf_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_veqhf_and : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_veqhf_and_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_veqhf_or : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_veqhf_or_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_veqhf_xor : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
````
- **L2157 EN**: Declares TableGen def record `V6_vabs_qf32_qf32_128B`.
  **L2157 CN**: 声明 TableGen def 记录 `V6_vabs_qf32_qf32_128B`。
- **L2158 EN**: Declares TableGen def record `V6_vabs_qf32_sf`.
  **L2158 CN**: 声明 TableGen def 记录 `V6_vabs_qf32_sf`。
- **L2159 EN**: Declares TableGen def record `V6_vabs_qf32_sf_128B`.
  **L2159 CN**: 声明 TableGen def 记录 `V6_vabs_qf32_sf_128B`。
- **L2160 EN**: Declares TableGen def record `V6_valign4`.
  **L2160 CN**: 声明 TableGen def 记录 `V6_valign4`。
- **L2161 EN**: Declares TableGen def record `V6_valign4_128B`.
  **L2161 CN**: 声明 TableGen def 记录 `V6_valign4_128B`。
- **L2162 EN**: Declares TableGen def record `V6_vconv_bf_qf32`.
  **L2162 CN**: 声明 TableGen def 记录 `V6_vconv_bf_qf32`。
- **L2163 EN**: Declares TableGen def record `V6_vconv_bf_qf32_128B`.
  **L2163 CN**: 声明 TableGen def 记录 `V6_vconv_bf_qf32_128B`。
- **L2164 EN**: Declares TableGen def record `V6_vconv_f8_qf16`.
  **L2164 CN**: 声明 TableGen def 记录 `V6_vconv_f8_qf16`。
- **L2165 EN**: Declares TableGen def record `V6_vconv_f8_qf16_128B`.
  **L2165 CN**: 声明 TableGen def 记录 `V6_vconv_f8_qf16_128B`。
- **L2166 EN**: Declares TableGen def record `V6_vconv_h_hf_rnd`.
  **L2166 CN**: 声明 TableGen def 记录 `V6_vconv_h_hf_rnd`。
- **L2167 EN**: Declares TableGen def record `V6_vconv_h_hf_rnd_128B`.
  **L2167 CN**: 声明 TableGen def 记录 `V6_vconv_h_hf_rnd_128B`。
- **L2168 EN**: Declares TableGen def record `V6_vconv_qf16_f8`.
  **L2168 CN**: 声明 TableGen def 记录 `V6_vconv_qf16_f8`。
- **L2169 EN**: Declares TableGen def record `V6_vconv_qf16_f8_128B`.
  **L2169 CN**: 声明 TableGen def 记录 `V6_vconv_qf16_f8_128B`。
- **L2170 EN**: Declares TableGen def record `V6_vconv_qf16_hf`.
  **L2170 CN**: 声明 TableGen def 记录 `V6_vconv_qf16_hf`。
- **L2171 EN**: Declares TableGen def record `V6_vconv_qf16_hf_128B`.
  **L2171 CN**: 声明 TableGen def 记录 `V6_vconv_qf16_hf_128B`。
- **L2172 EN**: Declares TableGen def record `V6_vconv_qf16_qf16`.
  **L2172 CN**: 声明 TableGen def 记录 `V6_vconv_qf16_qf16`。
- **L2173 EN**: Declares TableGen def record `V6_vconv_qf16_qf16_128B`.
  **L2173 CN**: 声明 TableGen def 记录 `V6_vconv_qf16_qf16_128B`。
- **L2174 EN**: Declares TableGen def record `V6_vconv_qf32_qf32`.
  **L2174 CN**: 声明 TableGen def 记录 `V6_vconv_qf32_qf32`。
- **L2175 EN**: Declares TableGen def record `V6_vconv_qf32_qf32_128B`.
  **L2175 CN**: 声明 TableGen def 记录 `V6_vconv_qf32_qf32_128B`。
- **L2176 EN**: Declares TableGen def record `V6_vconv_qf32_sf`.
  **L2176 CN**: 声明 TableGen def 记录 `V6_vconv_qf32_sf`。
- **L2177 EN**: Declares TableGen def record `V6_vconv_qf32_sf_128B`.
  **L2177 CN**: 声明 TableGen def 记录 `V6_vconv_qf32_sf_128B`。
- **L2178 EN**: Declares TableGen def record `V6_veqhf`.
  **L2178 CN**: 声明 TableGen def 记录 `V6_veqhf`。
- **L2179 EN**: Declares TableGen def record `V6_veqhf_128B`.
  **L2179 CN**: 声明 TableGen def 记录 `V6_veqhf_128B`。
- **L2180 EN**: Declares TableGen def record `V6_veqhf_and`.
  **L2180 CN**: 声明 TableGen def 记录 `V6_veqhf_and`。
- **L2181 EN**: Declares TableGen def record `V6_veqhf_and_128B`.
  **L2181 CN**: 声明 TableGen def 记录 `V6_veqhf_and_128B`。
- **L2182 EN**: Declares TableGen def record `V6_veqhf_or`.
  **L2182 CN**: 声明 TableGen def 记录 `V6_veqhf_or`。
- **L2183 EN**: Declares TableGen def record `V6_veqhf_or_128B`.
  **L2183 CN**: 声明 TableGen def 记录 `V6_veqhf_or_128B`。
- **L2184 EN**: Declares TableGen def record `V6_veqhf_xor`.
  **L2184 CN**: 声明 TableGen def 记录 `V6_veqhf_xor`。

### Lines 2185-2212

````tablegen
  def V6_veqhf_xor_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_veqsf : HexagonBuiltin<"_Vector<64, bool>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_veqsf_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_veqsf_and : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_veqsf_and_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_veqsf_or : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_veqsf_or_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_veqsf_xor : HexagonBuiltin<"_Vector<64, bool>(_Vector<64, bool>, _Vector<16, int>, _Vector<16, int>)">;
  def V6_veqsf_xor_128B : HexagonBuiltin<"_Vector<128, bool>(_Vector<128, bool>, _Vector<32, int>, _Vector<32, int>)">;
  def V6_vilog2_hf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vilog2_hf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vilog2_qf16 : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vilog2_qf16_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vilog2_qf32 : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vilog2_qf32_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vilog2_sf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vilog2_sf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vneg_qf16_hf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vneg_qf16_hf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vneg_qf16_qf16 : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vneg_qf16_qf16_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vneg_qf32_qf32 : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vneg_qf32_qf32_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vneg_qf32_sf : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>)">;
  def V6_vneg_qf32_sf_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>)">;
  def V6_vsub_hf_mix : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
  def V6_vsub_hf_mix_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
  def V6_vsub_sf_mix : HexagonBuiltin<"_Vector<16, int>(_Vector<16, int>, _Vector<16, int>)">;
````
- **L2185 EN**: Declares TableGen def record `V6_veqhf_xor_128B`.
  **L2185 CN**: 声明 TableGen def 记录 `V6_veqhf_xor_128B`。
- **L2186 EN**: Declares TableGen def record `V6_veqsf`.
  **L2186 CN**: 声明 TableGen def 记录 `V6_veqsf`。
- **L2187 EN**: Declares TableGen def record `V6_veqsf_128B`.
  **L2187 CN**: 声明 TableGen def 记录 `V6_veqsf_128B`。
- **L2188 EN**: Declares TableGen def record `V6_veqsf_and`.
  **L2188 CN**: 声明 TableGen def 记录 `V6_veqsf_and`。
- **L2189 EN**: Declares TableGen def record `V6_veqsf_and_128B`.
  **L2189 CN**: 声明 TableGen def 记录 `V6_veqsf_and_128B`。
- **L2190 EN**: Declares TableGen def record `V6_veqsf_or`.
  **L2190 CN**: 声明 TableGen def 记录 `V6_veqsf_or`。
- **L2191 EN**: Declares TableGen def record `V6_veqsf_or_128B`.
  **L2191 CN**: 声明 TableGen def 记录 `V6_veqsf_or_128B`。
- **L2192 EN**: Declares TableGen def record `V6_veqsf_xor`.
  **L2192 CN**: 声明 TableGen def 记录 `V6_veqsf_xor`。
- **L2193 EN**: Declares TableGen def record `V6_veqsf_xor_128B`.
  **L2193 CN**: 声明 TableGen def 记录 `V6_veqsf_xor_128B`。
- **L2194 EN**: Declares TableGen def record `V6_vilog2_hf`.
  **L2194 CN**: 声明 TableGen def 记录 `V6_vilog2_hf`。
- **L2195 EN**: Declares TableGen def record `V6_vilog2_hf_128B`.
  **L2195 CN**: 声明 TableGen def 记录 `V6_vilog2_hf_128B`。
- **L2196 EN**: Declares TableGen def record `V6_vilog2_qf16`.
  **L2196 CN**: 声明 TableGen def 记录 `V6_vilog2_qf16`。
- **L2197 EN**: Declares TableGen def record `V6_vilog2_qf16_128B`.
  **L2197 CN**: 声明 TableGen def 记录 `V6_vilog2_qf16_128B`。
- **L2198 EN**: Declares TableGen def record `V6_vilog2_qf32`.
  **L2198 CN**: 声明 TableGen def 记录 `V6_vilog2_qf32`。
- **L2199 EN**: Declares TableGen def record `V6_vilog2_qf32_128B`.
  **L2199 CN**: 声明 TableGen def 记录 `V6_vilog2_qf32_128B`。
- **L2200 EN**: Declares TableGen def record `V6_vilog2_sf`.
  **L2200 CN**: 声明 TableGen def 记录 `V6_vilog2_sf`。
- **L2201 EN**: Declares TableGen def record `V6_vilog2_sf_128B`.
  **L2201 CN**: 声明 TableGen def 记录 `V6_vilog2_sf_128B`。
- **L2202 EN**: Declares TableGen def record `V6_vneg_qf16_hf`.
  **L2202 CN**: 声明 TableGen def 记录 `V6_vneg_qf16_hf`。
- **L2203 EN**: Declares TableGen def record `V6_vneg_qf16_hf_128B`.
  **L2203 CN**: 声明 TableGen def 记录 `V6_vneg_qf16_hf_128B`。
- **L2204 EN**: Declares TableGen def record `V6_vneg_qf16_qf16`.
  **L2204 CN**: 声明 TableGen def 记录 `V6_vneg_qf16_qf16`。
- **L2205 EN**: Declares TableGen def record `V6_vneg_qf16_qf16_128B`.
  **L2205 CN**: 声明 TableGen def 记录 `V6_vneg_qf16_qf16_128B`。
- **L2206 EN**: Declares TableGen def record `V6_vneg_qf32_qf32`.
  **L2206 CN**: 声明 TableGen def 记录 `V6_vneg_qf32_qf32`。
- **L2207 EN**: Declares TableGen def record `V6_vneg_qf32_qf32_128B`.
  **L2207 CN**: 声明 TableGen def 记录 `V6_vneg_qf32_qf32_128B`。
- **L2208 EN**: Declares TableGen def record `V6_vneg_qf32_sf`.
  **L2208 CN**: 声明 TableGen def 记录 `V6_vneg_qf32_sf`。
- **L2209 EN**: Declares TableGen def record `V6_vneg_qf32_sf_128B`.
  **L2209 CN**: 声明 TableGen def 记录 `V6_vneg_qf32_sf_128B`。
- **L2210 EN**: Declares TableGen def record `V6_vsub_hf_mix`.
  **L2210 CN**: 声明 TableGen def 记录 `V6_vsub_hf_mix`。
- **L2211 EN**: Declares TableGen def record `V6_vsub_hf_mix_128B`.
  **L2211 CN**: 声明 TableGen def 记录 `V6_vsub_hf_mix_128B`。
- **L2212 EN**: Declares TableGen def record `V6_vsub_sf_mix`.
  **L2212 CN**: 声明 TableGen def 记录 `V6_vsub_sf_mix`。

### Lines 2213-2214

````tablegen
  def V6_vsub_sf_mix_128B : HexagonBuiltin<"_Vector<32, int>(_Vector<32, int>, _Vector<32, int>)">;
}
````
- **L2213 EN**: Declares TableGen def record `V6_vsub_sf_mix_128B`.
  **L2213 CN**: 声明 TableGen def 记录 `V6_vsub_sf_mix_128B`。
- **L2214 EN**: Closes the current lexical scope or compound statement.
  **L2214 CN**: 结束当前词法作用域或复合语句块。

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
- **Type-system support / 类型系统支撑**
  - **EN**: Provides metadata or helpers used to model Clang types and related dispatch logic.
  - **CN**: 提供建模 Clang 类型及相关分发逻辑所需的元数据或辅助工具。
- **Record families / 记录族**
  - **EN**: Groups related TableGen records so backends can derive structured generated output.
  - **CN**: 将相关 TableGen 记录组织成族，以便后端导出结构化的生成结果。

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: None detected. / 未检测到。
- **Macros / 宏**: No prominent macros detected. / 未检测到明显宏。
- **Types / 类型**: `VFeatures`, `V`, `HVXVFeatures`, `HVXV`, `HexagonBuiltin`, `HexagonBuiltinNoPrefix`
- **Functions or callables / 函数或可调用对象**: `strconcat`, `int`, `const`, `void`, `int>`, `double`, `float`, `bool>`
- **TableGen records / TableGen 记录**: `VFeatures`, `V`, `V79`, `V75`, `V73`, `V71`, `V69`, `V68`, `V67`, `V66`, `V65`, `V62`, `V60`, `V55`, `V5`, `HVXVFeatures`
- **Namespaces / 命名空间**: No explicit namespaces detected. / 未检测到显式命名空间。
