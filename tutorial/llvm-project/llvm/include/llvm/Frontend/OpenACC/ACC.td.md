# ACC.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/include/llvm/Frontend/OpenACC/ACC.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: This is the definition file for OpenACC 3.3 directives and clauses.
- **Purpose (CN)**: 该 TableGen 文件位于 `llvm/include/llvm/Frontend/OpenACC`，主要定义与 `ACC` 相关的记录、生成元数据和声明式规则。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````tablegen
//===-- ACC.td - OpenACC directive definition file ---------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This is the definition file for OpenACC 3.3 directives and clauses.
//
//===----------------------------------------------------------------------===//

include "llvm/Frontend/Directive/DirectiveBase.td"

//===----------------------------------------------------------------------===//
// Definition of general OpenACC information
//===----------------------------------------------------------------------===//

def OpenACC : DirectiveLanguage {
  let name = "OpenACC";
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
- **L9 EN**: Comment explains nearby logic, invariants, or intent: `This is the definition file for OpenACC 3.3 directives and clauses.`.
  **L9 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This is the definition file for OpenACC 3.3 directives and clauses.`。
- **L10 EN**: Separator comment used for visual grouping.
  **L10 CN**: 用于视觉分组的分隔注释。
- **L11 EN**: Banner comment marking a file or section boundary.
  **L11 CN**: 横幅注释，用于标记文件或章节边界。
- **L12 EN**: Blank line separating nearby declarations or logic blocks.
  **L12 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L13 EN**: Imports TableGen file "llvm/Frontend/Directive/DirectiveBase.td" to reuse frontend-facing integration helpers.
  **L13 CN**: 导入 TableGen 文件 "llvm/Frontend/Directive/DirectiveBase.td" 以复用面向前端的集成辅助组件。
- **L14 EN**: Blank line separating nearby declarations or logic blocks.
  **L14 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L15 EN**: Banner comment marking a file or section boundary.
  **L15 CN**: 横幅注释，用于标记文件或章节边界。
- **L16 EN**: Comment explains nearby logic, invariants, or intent: `Definition of general OpenACC information`.
  **L16 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Definition of general OpenACC information`。
- **L17 EN**: Banner comment marking a file or section boundary.
  **L17 CN**: 横幅注释，用于标记文件或章节边界。
- **L18 EN**: Blank line separating nearby declarations or logic blocks.
  **L18 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L19 EN**: Declares TableGen def `OpenACC`.
  **L19 CN**: 声明 TableGen def `OpenACC`。
- **L20 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L20 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 21-40

````tablegen
  let cppNamespace = "acc"; // final namespace will be llvm::acc
  let directivePrefix = "ACCD_";
  let clausePrefix = "ACCC_";
  let makeEnumAvailableInNamespace = true;
  let enableBitmaskEnumInNamespace = true;
  let clauseEnumSetClass = "AccClauseSet";
  let flangClauseBaseClass = "AccClause";
}

//===----------------------------------------------------------------------===//
// Definition of OpenACC clauses
//===----------------------------------------------------------------------===//

// 2.16.1
def ACCC_Async : Clause<[Spelling<"async">]> {
  let flangClass = "ScalarIntExpr";
  let isValueOptional = true;
}

// 2.9.7
````
- **L21 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L21 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L22 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L22 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L23 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L23 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L24 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L24 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L25 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L25 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L26 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L26 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L27 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L27 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L28 EN**: Closes the current lexical scope or compound statement.
  **L28 CN**: 结束当前词法作用域或复合语句块。
- **L29 EN**: Blank line separating nearby declarations or logic blocks.
  **L29 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L30 EN**: Banner comment marking a file or section boundary.
  **L30 CN**: 横幅注释，用于标记文件或章节边界。
- **L31 EN**: Comment explains nearby logic, invariants, or intent: `Definition of OpenACC clauses`.
  **L31 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Definition of OpenACC clauses`。
- **L32 EN**: Banner comment marking a file or section boundary.
  **L32 CN**: 横幅注释，用于标记文件或章节边界。
- **L33 EN**: Blank line separating nearby declarations or logic blocks.
  **L33 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L34 EN**: Comment explains nearby logic, invariants, or intent: `2.16.1`.
  **L34 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.16.1`。
- **L35 EN**: Declares TableGen def `ACCC_Async`.
  **L35 CN**: 声明 TableGen def `ACCC_Async`。
- **L36 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L36 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L37 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L37 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L38 EN**: Closes the current lexical scope or compound statement.
  **L38 CN**: 结束当前词法作用域或复合语句块。
- **L39 EN**: Blank line separating nearby declarations or logic blocks.
  **L39 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L40 EN**: Comment explains nearby logic, invariants, or intent: `2.9.7`.
  **L40 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.9.7`。

### Lines 41-60

````tablegen
def ACCC_Auto : Clause<[Spelling<"auto">]> {}

// 2.7.12
def ACCC_Attach : Clause<[Spelling<"attach">]> {
  let flangClass = "AccObjectList";
}

// 2.15.1
def ACCC_Bind : Clause<[Spelling<"bind">]> {
  let flangClass = "AccBindClause";
}

// 2.12
def ACCC_Capture : Clause<[Spelling<"capture">]> {
}

// 2.9.1
def ACCC_Collapse : Clause<[Spelling<"collapse">]> {
  let flangClass = "AccCollapseArg";
}
````
- **L41 EN**: Declares TableGen def `ACCC_Auto`.
  **L41 CN**: 声明 TableGen def `ACCC_Auto`。
- **L42 EN**: Blank line separating nearby declarations or logic blocks.
  **L42 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L43 EN**: Comment explains nearby logic, invariants, or intent: `2.7.12`.
  **L43 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.7.12`。
- **L44 EN**: Declares TableGen def `ACCC_Attach`.
  **L44 CN**: 声明 TableGen def `ACCC_Attach`。
- **L45 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L45 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L46 EN**: Closes the current lexical scope or compound statement.
  **L46 CN**: 结束当前词法作用域或复合语句块。
- **L47 EN**: Blank line separating nearby declarations or logic blocks.
  **L47 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L48 EN**: Comment explains nearby logic, invariants, or intent: `2.15.1`.
  **L48 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.15.1`。
- **L49 EN**: Declares TableGen def `ACCC_Bind`.
  **L49 CN**: 声明 TableGen def `ACCC_Bind`。
- **L50 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L50 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L51 EN**: Closes the current lexical scope or compound statement.
  **L51 CN**: 结束当前词法作用域或复合语句块。
- **L52 EN**: Blank line separating nearby declarations or logic blocks.
  **L52 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L53 EN**: Comment explains nearby logic, invariants, or intent: `2.12`.
  **L53 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.12`。
- **L54 EN**: Declares TableGen def `ACCC_Capture`.
  **L54 CN**: 声明 TableGen def `ACCC_Capture`。
- **L55 EN**: Closes the current lexical scope or compound statement.
  **L55 CN**: 结束当前词法作用域或复合语句块。
- **L56 EN**: Blank line separating nearby declarations or logic blocks.
  **L56 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L57 EN**: Comment explains nearby logic, invariants, or intent: `2.9.1`.
  **L57 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.9.1`。
- **L58 EN**: Declares TableGen def `ACCC_Collapse`.
  **L58 CN**: 声明 TableGen def `ACCC_Collapse`。
- **L59 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L59 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L60 EN**: Closes the current lexical scope or compound statement.
  **L60 CN**: 结束当前词法作用域或复合语句块。

### Lines 61-80

````tablegen

// 2.7.6
def ACCC_Copy
    : Clause<[Spelling<"copy">, Spelling<"present_or_copy">,
              Spelling<"pcopy">]> {
  let flangClass = "AccObjectList";
}

// 2.7.7
def ACCC_Copyin : Clause<[Spelling<"copyin">, Spelling<"present_or_copyin">,
                          Spelling<"pcopyin">]> {
  let flangClass = "AccObjectListWithModifier";
  let clangAccSpelling = "CopyIn";
}

// 2.7.8
def ACCC_Copyout : Clause<[Spelling<"copyout">, Spelling<"present_or_copyout">,
                           Spelling<"pcopyout">]> {
  let flangClass = "AccObjectListWithModifier";
  let clangAccSpelling = "CopyOut";
````
- **L61 EN**: Blank line separating nearby declarations or logic blocks.
  **L61 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L62 EN**: Comment explains nearby logic, invariants, or intent: `2.7.6`.
  **L62 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.7.6`。
- **L63 EN**: Declares TableGen def `ACCC_Copy`.
  **L63 CN**: 声明 TableGen def `ACCC_Copy`。
- **L64 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Clause<[Spelling<"copy">, Spelling<"present_or_copy">,`.
  **L64 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Clause<[Spelling<"copy">, Spelling<"present_or_copy">,`。
- **L65 EN**: Continues the surrounding expression or declaration: `Spelling<"pcopy">]> {`.
  **L65 CN**: 继续构造周围的表达式或声明：`Spelling<"pcopy">]> {`。
- **L66 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L66 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L67 EN**: Closes the current lexical scope or compound statement.
  **L67 CN**: 结束当前词法作用域或复合语句块。
- **L68 EN**: Blank line separating nearby declarations or logic blocks.
  **L68 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L69 EN**: Comment explains nearby logic, invariants, or intent: `2.7.7`.
  **L69 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.7.7`。
- **L70 EN**: Declares TableGen def `ACCC_Copyin`.
  **L70 CN**: 声明 TableGen def `ACCC_Copyin`。
- **L71 EN**: Continues the surrounding expression or declaration: `Spelling<"pcopyin">]> {`.
  **L71 CN**: 继续构造周围的表达式或声明：`Spelling<"pcopyin">]> {`。
- **L72 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L72 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L73 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L73 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L74 EN**: Closes the current lexical scope or compound statement.
  **L74 CN**: 结束当前词法作用域或复合语句块。
- **L75 EN**: Blank line separating nearby declarations or logic blocks.
  **L75 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L76 EN**: Comment explains nearby logic, invariants, or intent: `2.7.8`.
  **L76 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.7.8`。
- **L77 EN**: Declares TableGen def `ACCC_Copyout`.
  **L77 CN**: 声明 TableGen def `ACCC_Copyout`。
- **L78 EN**: Continues the surrounding expression or declaration: `Spelling<"pcopyout">]> {`.
  **L78 CN**: 继续构造周围的表达式或声明：`Spelling<"pcopyout">]> {`。
- **L79 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L79 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L80 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L80 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 81-100

````tablegen
}

// 2.7.9
def ACCC_Create : Clause<[Spelling<"create">, Spelling<"present_or_create">,
                          Spelling<"pcreate">]> {
  let flangClass = "AccObjectListWithModifier";
}

// 2.5.16
def ACC_Default_none : EnumVal<"none", 1, 1> { let isDefault = 1; }
def ACC_Default_present : EnumVal<"present", 0, 1> {}

def ACCC_Default : Clause<[Spelling<"default">]> {
  let flangClass = "AccDefaultClause";
  let enumClauseValue = "DefaultValue";
  let allowedClauseValues = [
    ACC_Default_present,
    ACC_Default_none
  ];
}
````
- **L81 EN**: Closes the current lexical scope or compound statement.
  **L81 CN**: 结束当前词法作用域或复合语句块。
- **L82 EN**: Blank line separating nearby declarations or logic blocks.
  **L82 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L83 EN**: Comment explains nearby logic, invariants, or intent: `2.7.9`.
  **L83 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.7.9`。
- **L84 EN**: Declares TableGen def `ACCC_Create`.
  **L84 CN**: 声明 TableGen def `ACCC_Create`。
- **L85 EN**: Continues the surrounding expression or declaration: `Spelling<"pcreate">]> {`.
  **L85 CN**: 继续构造周围的表达式或声明：`Spelling<"pcreate">]> {`。
- **L86 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L86 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L87 EN**: Closes the current lexical scope or compound statement.
  **L87 CN**: 结束当前词法作用域或复合语句块。
- **L88 EN**: Blank line separating nearby declarations or logic blocks.
  **L88 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L89 EN**: Comment explains nearby logic, invariants, or intent: `2.5.16`.
  **L89 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.5.16`。
- **L90 EN**: Declares TableGen def `ACC_Default_none`.
  **L90 CN**: 声明 TableGen def `ACC_Default_none`。
- **L91 EN**: Declares TableGen def `ACC_Default_present`.
  **L91 CN**: 声明 TableGen def `ACC_Default_present`。
- **L92 EN**: Blank line separating nearby declarations or logic blocks.
  **L92 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L93 EN**: Declares TableGen def `ACCC_Default`.
  **L93 CN**: 声明 TableGen def `ACCC_Default`。
- **L94 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L94 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L95 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L95 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L96 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L96 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L97 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ACC_Default_present,`.
  **L97 CN**: 继续一个多行参数列表、初始化器或聚合项：`ACC_Default_present,`。
- **L98 EN**: Continues the surrounding expression or declaration: `ACC_Default_none`.
  **L98 CN**: 继续构造周围的表达式或声明：`ACC_Default_none`。
- **L99 EN**: Executes a standalone statement or declaration: `];`.
  **L99 CN**: 执行一条独立语句或声明：`];`。
- **L100 EN**: Closes the current lexical scope or compound statement.
  **L100 CN**: 结束当前词法作用域或复合语句块。

### Lines 101-120

````tablegen

// 2.14.3
def ACCC_DefaultAsync : Clause<[Spelling<"default_async">]> {
  let flangClass = "ScalarIntExpr";
}

// 2.7.11
def ACCC_Delete : Clause<[Spelling<"delete">]> {
  let flangClass = "AccObjectList";
}

// 2.7.13
def ACCC_Detach : Clause<[Spelling<"detach">]> {
  let flangClass = "AccObjectList";
}

// 2.14.4
def ACCC_Device : Clause<[Spelling<"device">]> {
  let flangClass = "AccObjectList";
}
````
- **L101 EN**: Blank line separating nearby declarations or logic blocks.
  **L101 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L102 EN**: Comment explains nearby logic, invariants, or intent: `2.14.3`.
  **L102 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.14.3`。
- **L103 EN**: Declares TableGen def `ACCC_DefaultAsync`.
  **L103 CN**: 声明 TableGen def `ACCC_DefaultAsync`。
- **L104 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L104 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L105 EN**: Closes the current lexical scope or compound statement.
  **L105 CN**: 结束当前词法作用域或复合语句块。
- **L106 EN**: Blank line separating nearby declarations or logic blocks.
  **L106 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L107 EN**: Comment explains nearby logic, invariants, or intent: `2.7.11`.
  **L107 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.7.11`。
- **L108 EN**: Declares TableGen def `ACCC_Delete`.
  **L108 CN**: 声明 TableGen def `ACCC_Delete`。
- **L109 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L109 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L110 EN**: Closes the current lexical scope or compound statement.
  **L110 CN**: 结束当前词法作用域或复合语句块。
- **L111 EN**: Blank line separating nearby declarations or logic blocks.
  **L111 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L112 EN**: Comment explains nearby logic, invariants, or intent: `2.7.13`.
  **L112 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.7.13`。
- **L113 EN**: Declares TableGen def `ACCC_Detach`.
  **L113 CN**: 声明 TableGen def `ACCC_Detach`。
- **L114 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L114 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L115 EN**: Closes the current lexical scope or compound statement.
  **L115 CN**: 结束当前词法作用域或复合语句块。
- **L116 EN**: Blank line separating nearby declarations or logic blocks.
  **L116 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L117 EN**: Comment explains nearby logic, invariants, or intent: `2.14.4`.
  **L117 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.14.4`。
- **L118 EN**: Declares TableGen def `ACCC_Device`.
  **L118 CN**: 声明 TableGen def `ACCC_Device`。
- **L119 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L119 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L120 EN**: Closes the current lexical scope or compound statement.
  **L120 CN**: 结束当前词法作用域或复合语句块。

### Lines 121-140

````tablegen

// 2.14.1 - 2.14.2
def ACCC_DeviceNum : Clause<[Spelling<"device_num">]>  {
  let flangClass = "ScalarIntExpr";
}

// 2.7.4
def ACCC_DevicePtr : Clause<[Spelling<"deviceptr">]> {
  let flangClass = "AccObjectList";
  let clangAccSpelling = "DevicePtr";
}

// 2.13.1
def ACCC_DeviceResident : Clause<[Spelling<"device_resident">]> {
  let flangClass = "AccObjectList";
}

// 2.4
def ACCC_DeviceType : Clause<[Spelling<"device_type">, Spelling<"dtype">]> {
  let flangClass = "AccDeviceTypeExprList";
````
- **L121 EN**: Blank line separating nearby declarations or logic blocks.
  **L121 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L122 EN**: Comment explains nearby logic, invariants, or intent: `2.14.1 - 2.14.2`.
  **L122 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.14.1 - 2.14.2`。
- **L123 EN**: Declares TableGen def `ACCC_DeviceNum`.
  **L123 CN**: 声明 TableGen def `ACCC_DeviceNum`。
- **L124 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L124 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L125 EN**: Closes the current lexical scope or compound statement.
  **L125 CN**: 结束当前词法作用域或复合语句块。
- **L126 EN**: Blank line separating nearby declarations or logic blocks.
  **L126 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L127 EN**: Comment explains nearby logic, invariants, or intent: `2.7.4`.
  **L127 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.7.4`。
- **L128 EN**: Declares TableGen def `ACCC_DevicePtr`.
  **L128 CN**: 声明 TableGen def `ACCC_DevicePtr`。
- **L129 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L129 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L130 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L130 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L131 EN**: Closes the current lexical scope or compound statement.
  **L131 CN**: 结束当前词法作用域或复合语句块。
- **L132 EN**: Blank line separating nearby declarations or logic blocks.
  **L132 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L133 EN**: Comment explains nearby logic, invariants, or intent: `2.13.1`.
  **L133 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.13.1`。
- **L134 EN**: Declares TableGen def `ACCC_DeviceResident`.
  **L134 CN**: 声明 TableGen def `ACCC_DeviceResident`。
- **L135 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L135 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L136 EN**: Closes the current lexical scope or compound statement.
  **L136 CN**: 结束当前词法作用域或复合语句块。
- **L137 EN**: Blank line separating nearby declarations or logic blocks.
  **L137 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L138 EN**: Comment explains nearby logic, invariants, or intent: `2.4`.
  **L138 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.4`。
- **L139 EN**: Declares TableGen def `ACCC_DeviceType`.
  **L139 CN**: 声明 TableGen def `ACCC_DeviceType`。
- **L140 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L140 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 141-160

````tablegen
  let defaultValue = "*";
}

// 2.6.6
def ACCC_Finalize : Clause<[Spelling<"finalize">]> {}

// 2.5.14
def ACCC_FirstPrivate : Clause<[Spelling<"firstprivate">]> {
  let flangClass = "AccObjectList";
  let clangAccSpelling = "FirstPrivate";
}

// 2.9.2
def ACCC_Gang : Clause<[Spelling<"gang">]> {
  let flangClass = "AccGangArgList";
  let isValueOptional = true;
}

// 2.14.4
def ACCC_Host : Clause<[Spelling<"host">]> {
````
- **L141 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L141 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L142 EN**: Closes the current lexical scope or compound statement.
  **L142 CN**: 结束当前词法作用域或复合语句块。
- **L143 EN**: Blank line separating nearby declarations or logic blocks.
  **L143 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L144 EN**: Comment explains nearby logic, invariants, or intent: `2.6.6`.
  **L144 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.6.6`。
- **L145 EN**: Declares TableGen def `ACCC_Finalize`.
  **L145 CN**: 声明 TableGen def `ACCC_Finalize`。
- **L146 EN**: Blank line separating nearby declarations or logic blocks.
  **L146 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L147 EN**: Comment explains nearby logic, invariants, or intent: `2.5.14`.
  **L147 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.5.14`。
- **L148 EN**: Declares TableGen def `ACCC_FirstPrivate`.
  **L148 CN**: 声明 TableGen def `ACCC_FirstPrivate`。
- **L149 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L149 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L150 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L150 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L151 EN**: Closes the current lexical scope or compound statement.
  **L151 CN**: 结束当前词法作用域或复合语句块。
- **L152 EN**: Blank line separating nearby declarations or logic blocks.
  **L152 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L153 EN**: Comment explains nearby logic, invariants, or intent: `2.9.2`.
  **L153 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.9.2`。
- **L154 EN**: Declares TableGen def `ACCC_Gang`.
  **L154 CN**: 声明 TableGen def `ACCC_Gang`。
- **L155 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L155 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L156 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L156 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L157 EN**: Closes the current lexical scope or compound statement.
  **L157 CN**: 结束当前词法作用域或复合语句块。
- **L158 EN**: Blank line separating nearby declarations or logic blocks.
  **L158 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L159 EN**: Comment explains nearby logic, invariants, or intent: `2.14.4`.
  **L159 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.14.4`。
- **L160 EN**: Declares TableGen def `ACCC_Host`.
  **L160 CN**: 声明 TableGen def `ACCC_Host`。

### Lines 161-180

````tablegen
  let flangClass = "AccObjectList";
}

// 2.5.6
def ACCC_If : Clause<[Spelling<"if">]> {
  let flangClass = "ScalarExpr";
}

// 2.14.4
def ACCC_IfPresent : Clause<[Spelling<"if_present">]> {}

// 2.9.6
def ACCC_Independent : Clause<[Spelling<"independent">]> {}

// 2.13.3
def ACCC_Link : Clause<[Spelling<"link">]> {
  let flangClass = "AccObjectList";
}

// 2.7.10
````
- **L161 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L161 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L162 EN**: Closes the current lexical scope or compound statement.
  **L162 CN**: 结束当前词法作用域或复合语句块。
- **L163 EN**: Blank line separating nearby declarations or logic blocks.
  **L163 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L164 EN**: Comment explains nearby logic, invariants, or intent: `2.5.6`.
  **L164 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.5.6`。
- **L165 EN**: Declares TableGen def `ACCC_If`.
  **L165 CN**: 声明 TableGen def `ACCC_If`。
- **L166 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L166 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L167 EN**: Closes the current lexical scope or compound statement.
  **L167 CN**: 结束当前词法作用域或复合语句块。
- **L168 EN**: Blank line separating nearby declarations or logic blocks.
  **L168 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L169 EN**: Comment explains nearby logic, invariants, or intent: `2.14.4`.
  **L169 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.14.4`。
- **L170 EN**: Declares TableGen def `ACCC_IfPresent`.
  **L170 CN**: 声明 TableGen def `ACCC_IfPresent`。
- **L171 EN**: Blank line separating nearby declarations or logic blocks.
  **L171 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L172 EN**: Comment explains nearby logic, invariants, or intent: `2.9.6`.
  **L172 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.9.6`。
- **L173 EN**: Declares TableGen def `ACCC_Independent`.
  **L173 CN**: 声明 TableGen def `ACCC_Independent`。
- **L174 EN**: Blank line separating nearby declarations or logic blocks.
  **L174 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L175 EN**: Comment explains nearby logic, invariants, or intent: `2.13.3`.
  **L175 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.13.3`。
- **L176 EN**: Declares TableGen def `ACCC_Link`.
  **L176 CN**: 声明 TableGen def `ACCC_Link`。
- **L177 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L177 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L178 EN**: Closes the current lexical scope or compound statement.
  **L178 CN**: 结束当前词法作用域或复合语句块。
- **L179 EN**: Blank line separating nearby declarations or logic blocks.
  **L179 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L180 EN**: Comment explains nearby logic, invariants, or intent: `2.7.10`.
  **L180 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.7.10`。

### Lines 181-200

````tablegen
def ACCC_NoCreate : Clause<[Spelling<"no_create">]> {
  let flangClass = "AccObjectList";
}

// 2.15.1
def ACCC_NoHost : Clause<[Spelling<"nohost">]> {
  let clangAccSpelling = "NoHost";
}

// 2.5.10
def ACCC_NumGangs : Clause<[Spelling<"num_gangs">]> {
  let flangClass = "ScalarIntExpr";
  let isValueList = 1;
}

// 2.5.11
def ACCC_NumWorkers : Clause<[Spelling<"num_workers">]> {
  let flangClass = "ScalarIntExpr";
}

````
- **L181 EN**: Declares TableGen def `ACCC_NoCreate`.
  **L181 CN**: 声明 TableGen def `ACCC_NoCreate`。
- **L182 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L182 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L183 EN**: Closes the current lexical scope or compound statement.
  **L183 CN**: 结束当前词法作用域或复合语句块。
- **L184 EN**: Blank line separating nearby declarations or logic blocks.
  **L184 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L185 EN**: Comment explains nearby logic, invariants, or intent: `2.15.1`.
  **L185 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.15.1`。
- **L186 EN**: Declares TableGen def `ACCC_NoHost`.
  **L186 CN**: 声明 TableGen def `ACCC_NoHost`。
- **L187 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L187 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L188 EN**: Closes the current lexical scope or compound statement.
  **L188 CN**: 结束当前词法作用域或复合语句块。
- **L189 EN**: Blank line separating nearby declarations or logic blocks.
  **L189 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L190 EN**: Comment explains nearby logic, invariants, or intent: `2.5.10`.
  **L190 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.5.10`。
- **L191 EN**: Declares TableGen def `ACCC_NumGangs`.
  **L191 CN**: 声明 TableGen def `ACCC_NumGangs`。
- **L192 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L192 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L193 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L193 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L194 EN**: Closes the current lexical scope or compound statement.
  **L194 CN**: 结束当前词法作用域或复合语句块。
- **L195 EN**: Blank line separating nearby declarations or logic blocks.
  **L195 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L196 EN**: Comment explains nearby logic, invariants, or intent: `2.5.11`.
  **L196 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.5.11`。
- **L197 EN**: Declares TableGen def `ACCC_NumWorkers`.
  **L197 CN**: 声明 TableGen def `ACCC_NumWorkers`。
- **L198 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L198 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L199 EN**: Closes the current lexical scope or compound statement.
  **L199 CN**: 结束当前词法作用域或复合语句块。
- **L200 EN**: Blank line separating nearby declarations or logic blocks.
  **L200 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 201-220

````tablegen
// 2.7.5
def ACCC_Present : Clause<[Spelling<"present">]> {
  let flangClass = "AccObjectList";
}

// 2.5.13
def ACCC_Private : Clause<[Spelling<"private">]> {
  let flangClass = "AccObjectList";
}

// 2.9.8
def ACCC_Tile : Clause<[Spelling<"tile">]> {
  let flangClass = "AccTileExprList";
}

// 2.8.1
def ACCC_UseDevice : Clause<[Spelling<"use_device">]> {
  let flangClass = "AccObjectList";
}

````
- **L201 EN**: Comment explains nearby logic, invariants, or intent: `2.7.5`.
  **L201 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.7.5`。
- **L202 EN**: Declares TableGen def `ACCC_Present`.
  **L202 CN**: 声明 TableGen def `ACCC_Present`。
- **L203 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L203 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L204 EN**: Closes the current lexical scope or compound statement.
  **L204 CN**: 结束当前词法作用域或复合语句块。
- **L205 EN**: Blank line separating nearby declarations or logic blocks.
  **L205 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L206 EN**: Comment explains nearby logic, invariants, or intent: `2.5.13`.
  **L206 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.5.13`。
- **L207 EN**: Declares TableGen def `ACCC_Private`.
  **L207 CN**: 声明 TableGen def `ACCC_Private`。
- **L208 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L208 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L209 EN**: Closes the current lexical scope or compound statement.
  **L209 CN**: 结束当前词法作用域或复合语句块。
- **L210 EN**: Blank line separating nearby declarations or logic blocks.
  **L210 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L211 EN**: Comment explains nearby logic, invariants, or intent: `2.9.8`.
  **L211 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.9.8`。
- **L212 EN**: Declares TableGen def `ACCC_Tile`.
  **L212 CN**: 声明 TableGen def `ACCC_Tile`。
- **L213 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L213 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L214 EN**: Closes the current lexical scope or compound statement.
  **L214 CN**: 结束当前词法作用域或复合语句块。
- **L215 EN**: Blank line separating nearby declarations or logic blocks.
  **L215 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L216 EN**: Comment explains nearby logic, invariants, or intent: `2.8.1`.
  **L216 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.8.1`。
- **L217 EN**: Declares TableGen def `ACCC_UseDevice`.
  **L217 CN**: 声明 TableGen def `ACCC_UseDevice`。
- **L218 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L218 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L219 EN**: Closes the current lexical scope or compound statement.
  **L219 CN**: 结束当前词法作用域或复合语句块。
- **L220 EN**: Blank line separating nearby declarations or logic blocks.
  **L220 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 221-240

````tablegen
// 2.12
def ACCC_Read : Clause<[Spelling<"read">]> {}

// 2.5.15
def ACCC_Reduction : Clause<[Spelling<"reduction">]> {
  let flangClass = "AccObjectListWithReduction";
}

// 2.5.7
def ACCC_Self : Clause<[Spelling<"self">]> {
  let flangClass = "AccSelfClause";
  let isValueOptional = true;
}

// 2.9.5
def ACCC_Seq : Clause<[Spelling<"seq">]> {}

// Non-standard extension
def ACCC_ShortLoop : Clause<[Spelling<"shortloop">]> {}

````
- **L221 EN**: Comment explains nearby logic, invariants, or intent: `2.12`.
  **L221 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.12`。
- **L222 EN**: Declares TableGen def `ACCC_Read`.
  **L222 CN**: 声明 TableGen def `ACCC_Read`。
- **L223 EN**: Blank line separating nearby declarations or logic blocks.
  **L223 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L224 EN**: Comment explains nearby logic, invariants, or intent: `2.5.15`.
  **L224 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.5.15`。
- **L225 EN**: Declares TableGen def `ACCC_Reduction`.
  **L225 CN**: 声明 TableGen def `ACCC_Reduction`。
- **L226 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L226 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L227 EN**: Closes the current lexical scope or compound statement.
  **L227 CN**: 结束当前词法作用域或复合语句块。
- **L228 EN**: Blank line separating nearby declarations or logic blocks.
  **L228 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L229 EN**: Comment explains nearby logic, invariants, or intent: `2.5.7`.
  **L229 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.5.7`。
- **L230 EN**: Declares TableGen def `ACCC_Self`.
  **L230 CN**: 声明 TableGen def `ACCC_Self`。
- **L231 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L231 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L232 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L232 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L233 EN**: Closes the current lexical scope or compound statement.
  **L233 CN**: 结束当前词法作用域或复合语句块。
- **L234 EN**: Blank line separating nearby declarations or logic blocks.
  **L234 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L235 EN**: Comment explains nearby logic, invariants, or intent: `2.9.5`.
  **L235 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.9.5`。
- **L236 EN**: Declares TableGen def `ACCC_Seq`.
  **L236 CN**: 声明 TableGen def `ACCC_Seq`。
- **L237 EN**: Blank line separating nearby declarations or logic blocks.
  **L237 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L238 EN**: Comment explains nearby logic, invariants, or intent: `Non-standard extension`.
  **L238 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Non-standard extension`。
- **L239 EN**: Declares TableGen def `ACCC_ShortLoop`.
  **L239 CN**: 声明 TableGen def `ACCC_ShortLoop`。
- **L240 EN**: Blank line separating nearby declarations or logic blocks.
  **L240 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 241-260

````tablegen
// 2.9.4
def ACCC_Vector : Clause<[Spelling<"vector">]> {
  let flangClass = "ScalarIntExpr";
  let isValueOptional = true;
  let prefix = "length";
}

// 2.5.12
def ACCC_VectorLength : Clause<[Spelling<"vector_length">]> {
  let flangClass = "ScalarIntExpr";
}

// 2.16.2
def ACCC_Wait : Clause<[Spelling<"wait">]> {
  let flangClass = "AccWaitArgument";
  let isValueOptional = true;
}

// 2.9.3
def ACCC_Worker: Clause<[Spelling<"worker">]> {
````
- **L241 EN**: Comment explains nearby logic, invariants, or intent: `2.9.4`.
  **L241 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.9.4`。
- **L242 EN**: Declares TableGen def `ACCC_Vector`.
  **L242 CN**: 声明 TableGen def `ACCC_Vector`。
- **L243 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L243 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L244 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L244 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L245 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L245 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L246 EN**: Closes the current lexical scope or compound statement.
  **L246 CN**: 结束当前词法作用域或复合语句块。
- **L247 EN**: Blank line separating nearby declarations or logic blocks.
  **L247 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L248 EN**: Comment explains nearby logic, invariants, or intent: `2.5.12`.
  **L248 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.5.12`。
- **L249 EN**: Declares TableGen def `ACCC_VectorLength`.
  **L249 CN**: 声明 TableGen def `ACCC_VectorLength`。
- **L250 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L250 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L251 EN**: Closes the current lexical scope or compound statement.
  **L251 CN**: 结束当前词法作用域或复合语句块。
- **L252 EN**: Blank line separating nearby declarations or logic blocks.
  **L252 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L253 EN**: Comment explains nearby logic, invariants, or intent: `2.16.2`.
  **L253 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.16.2`。
- **L254 EN**: Declares TableGen def `ACCC_Wait`.
  **L254 CN**: 声明 TableGen def `ACCC_Wait`。
- **L255 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L255 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L256 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L256 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L257 EN**: Closes the current lexical scope or compound statement.
  **L257 CN**: 结束当前词法作用域或复合语句块。
- **L258 EN**: Blank line separating nearby declarations or logic blocks.
  **L258 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L259 EN**: Comment explains nearby logic, invariants, or intent: `2.9.3`.
  **L259 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.9.3`。
- **L260 EN**: Declares TableGen def `ACCC_Worker`.
  **L260 CN**: 声明 TableGen def `ACCC_Worker`。

### Lines 261-280

````tablegen
  let flangClass = "ScalarIntExpr";
  let isValueOptional = true;
  let prefix = "num";
}

// 2.12
def ACCC_Write : Clause<[Spelling<"write">]> {}

def ACCC_Unknown : Clause<[Spelling<"unknown">]> {
  let isDefault = true;
}

//===----------------------------------------------------------------------===//
// Definition of OpenACC directives
//===----------------------------------------------------------------------===//

// 2.12
def ACC_Atomic : Directive<[Spelling<"atomic">]> {
  let allowedOnceClauses = [VersionedClause<ACCC_If, 34>];
  let association = AS_Block;
````
- **L261 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L261 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L262 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L262 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L263 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L263 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L264 EN**: Closes the current lexical scope or compound statement.
  **L264 CN**: 结束当前词法作用域或复合语句块。
- **L265 EN**: Blank line separating nearby declarations or logic blocks.
  **L265 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L266 EN**: Comment explains nearby logic, invariants, or intent: `2.12`.
  **L266 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.12`。
- **L267 EN**: Declares TableGen def `ACCC_Write`.
  **L267 CN**: 声明 TableGen def `ACCC_Write`。
- **L268 EN**: Blank line separating nearby declarations or logic blocks.
  **L268 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L269 EN**: Declares TableGen def `ACCC_Unknown`.
  **L269 CN**: 声明 TableGen def `ACCC_Unknown`。
- **L270 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L270 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L271 EN**: Closes the current lexical scope or compound statement.
  **L271 CN**: 结束当前词法作用域或复合语句块。
- **L272 EN**: Blank line separating nearby declarations or logic blocks.
  **L272 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L273 EN**: Banner comment marking a file or section boundary.
  **L273 CN**: 横幅注释，用于标记文件或章节边界。
- **L274 EN**: Comment explains nearby logic, invariants, or intent: `Definition of OpenACC directives`.
  **L274 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Definition of OpenACC directives`。
- **L275 EN**: Banner comment marking a file or section boundary.
  **L275 CN**: 横幅注释，用于标记文件或章节边界。
- **L276 EN**: Blank line separating nearby declarations or logic blocks.
  **L276 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L277 EN**: Comment explains nearby logic, invariants, or intent: `2.12`.
  **L277 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.12`。
- **L278 EN**: Declares TableGen def `ACC_Atomic`.
  **L278 CN**: 声明 TableGen def `ACC_Atomic`。
- **L279 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L279 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L280 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L280 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 281-300

````tablegen
  let category = CA_Executable;
}

// 2.6.5
def ACC_Data : Directive<[Spelling<"data">]> {
  let allowedOnceClauses = [
    VersionedClause<ACCC_If>,
    VersionedClause<ACCC_Default>
  ];
  let allowedClauses = [
    VersionedClause<ACCC_Async, 32>,
    VersionedClause<ACCC_DeviceType, 32>,
    VersionedClause<ACCC_Wait, 32>
  ];
  let requiredClauses = [
    VersionedClause<ACCC_Attach>,
    VersionedClause<ACCC_Copy>,
    VersionedClause<ACCC_Copyin>,
    VersionedClause<ACCC_Copyout>,
    VersionedClause<ACCC_Create>,
````
- **L281 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L281 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L282 EN**: Closes the current lexical scope or compound statement.
  **L282 CN**: 结束当前词法作用域或复合语句块。
- **L283 EN**: Blank line separating nearby declarations or logic blocks.
  **L283 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L284 EN**: Comment explains nearby logic, invariants, or intent: `2.6.5`.
  **L284 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.6.5`。
- **L285 EN**: Declares TableGen def `ACC_Data`.
  **L285 CN**: 声明 TableGen def `ACC_Data`。
- **L286 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L286 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L287 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_If>,`.
  **L287 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_If>,`。
- **L288 EN**: Continues the surrounding expression or declaration: `VersionedClause<ACCC_Default>`.
  **L288 CN**: 继续构造周围的表达式或声明：`VersionedClause<ACCC_Default>`。
- **L289 EN**: Executes a standalone statement or declaration: `];`.
  **L289 CN**: 执行一条独立语句或声明：`];`。
- **L290 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L290 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L291 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Async, 32>,`.
  **L291 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Async, 32>,`。
- **L292 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_DeviceType, 32>,`.
  **L292 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_DeviceType, 32>,`。
- **L293 EN**: Continues the surrounding expression or declaration: `VersionedClause<ACCC_Wait, 32>`.
  **L293 CN**: 继续构造周围的表达式或声明：`VersionedClause<ACCC_Wait, 32>`。
- **L294 EN**: Executes a standalone statement or declaration: `];`.
  **L294 CN**: 执行一条独立语句或声明：`];`。
- **L295 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L295 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L296 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Attach>,`.
  **L296 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Attach>,`。
- **L297 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Copy>,`.
  **L297 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Copy>,`。
- **L298 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Copyin>,`.
  **L298 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Copyin>,`。
- **L299 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Copyout>,`.
  **L299 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Copyout>,`。
- **L300 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Create>,`.
  **L300 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Create>,`。

### Lines 301-320

````tablegen
    VersionedClause<ACCC_Default>,
    VersionedClause<ACCC_DevicePtr>,
    VersionedClause<ACCC_NoCreate>,
    VersionedClause<ACCC_Present>
  ];
  let association = AS_Block;
  let category = CA_Executable;
}

// 2.13
def ACC_Declare : Directive<[Spelling<"declare">]> {
  let allowedClauses = [
    VersionedClause<ACCC_Copy>,
    VersionedClause<ACCC_Copyin>,
    VersionedClause<ACCC_Copyout>,
    VersionedClause<ACCC_Create>,
    VersionedClause<ACCC_Present>,
    VersionedClause<ACCC_DevicePtr>,
    VersionedClause<ACCC_DeviceResident>,
    VersionedClause<ACCC_Link>
````
- **L301 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Default>,`.
  **L301 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Default>,`。
- **L302 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_DevicePtr>,`.
  **L302 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_DevicePtr>,`。
- **L303 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_NoCreate>,`.
  **L303 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_NoCreate>,`。
- **L304 EN**: Continues the surrounding expression or declaration: `VersionedClause<ACCC_Present>`.
  **L304 CN**: 继续构造周围的表达式或声明：`VersionedClause<ACCC_Present>`。
- **L305 EN**: Executes a standalone statement or declaration: `];`.
  **L305 CN**: 执行一条独立语句或声明：`];`。
- **L306 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L306 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L307 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L307 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L308 EN**: Closes the current lexical scope or compound statement.
  **L308 CN**: 结束当前词法作用域或复合语句块。
- **L309 EN**: Blank line separating nearby declarations or logic blocks.
  **L309 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L310 EN**: Comment explains nearby logic, invariants, or intent: `2.13`.
  **L310 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.13`。
- **L311 EN**: Declares TableGen def `ACC_Declare`.
  **L311 CN**: 声明 TableGen def `ACC_Declare`。
- **L312 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L312 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L313 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Copy>,`.
  **L313 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Copy>,`。
- **L314 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Copyin>,`.
  **L314 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Copyin>,`。
- **L315 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Copyout>,`.
  **L315 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Copyout>,`。
- **L316 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Create>,`.
  **L316 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Create>,`。
- **L317 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Present>,`.
  **L317 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Present>,`。
- **L318 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_DevicePtr>,`.
  **L318 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_DevicePtr>,`。
- **L319 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_DeviceResident>,`.
  **L319 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_DeviceResident>,`。
- **L320 EN**: Continues the surrounding expression or declaration: `VersionedClause<ACCC_Link>`.
  **L320 CN**: 继续构造周围的表达式或声明：`VersionedClause<ACCC_Link>`。

### Lines 321-340

````tablegen
  ];
  let association = AS_None;
  let category = CA_Declarative;
}

// 2.5.3
def ACC_Kernels : Directive<[Spelling<"kernels">]> {
  let allowedClauses = [VersionedClause<ACCC_Async>,
                        VersionedClause<ACCC_Attach>,
                        VersionedClause<ACCC_Copy>,
                        VersionedClause<ACCC_Copyin>,
                        VersionedClause<ACCC_Copyout>,
                        VersionedClause<ACCC_Create>,
                        VersionedClause<ACCC_DeviceType>,
                        VersionedClause<ACCC_NoCreate>,
                        VersionedClause<ACCC_NumGangs>,
                        VersionedClause<ACCC_NumWorkers>,
                        VersionedClause<ACCC_Present>,
                        VersionedClause<ACCC_DevicePtr>,
                        VersionedClause<ACCC_VectorLength>,
````
- **L321 EN**: Executes a standalone statement or declaration: `];`.
  **L321 CN**: 执行一条独立语句或声明：`];`。
- **L322 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L322 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L323 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L323 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L324 EN**: Closes the current lexical scope or compound statement.
  **L324 CN**: 结束当前词法作用域或复合语句块。
- **L325 EN**: Blank line separating nearby declarations or logic blocks.
  **L325 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L326 EN**: Comment explains nearby logic, invariants, or intent: `2.5.3`.
  **L326 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.5.3`。
- **L327 EN**: Declares TableGen def `ACC_Kernels`.
  **L327 CN**: 声明 TableGen def `ACC_Kernels`。
- **L328 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L328 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L329 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Attach>,`.
  **L329 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Attach>,`。
- **L330 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Copy>,`.
  **L330 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Copy>,`。
- **L331 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Copyin>,`.
  **L331 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Copyin>,`。
- **L332 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Copyout>,`.
  **L332 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Copyout>,`。
- **L333 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Create>,`.
  **L333 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Create>,`。
- **L334 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_DeviceType>,`.
  **L334 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_DeviceType>,`。
- **L335 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_NoCreate>,`.
  **L335 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_NoCreate>,`。
- **L336 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_NumGangs>,`.
  **L336 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_NumGangs>,`。
- **L337 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_NumWorkers>,`.
  **L337 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_NumWorkers>,`。
- **L338 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Present>,`.
  **L338 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Present>,`。
- **L339 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_DevicePtr>,`.
  **L339 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_DevicePtr>,`。
- **L340 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_VectorLength>,`.
  **L340 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_VectorLength>,`。

### Lines 341-360

````tablegen
                        VersionedClause<ACCC_Wait>];
  let allowedOnceClauses = [VersionedClause<ACCC_Default>,
                            VersionedClause<ACCC_If>,
                            VersionedClause<ACCC_Self>];
  let association = AS_Block;
  let category = CA_Executable;
}

// 2.5.1
def ACC_Parallel : Directive<[Spelling<"parallel">]> {
  let allowedClauses = [
    VersionedClause<ACCC_Attach>,
    VersionedClause<ACCC_Async>,
    VersionedClause<ACCC_Copy>,
    VersionedClause<ACCC_Copyin>,
    VersionedClause<ACCC_Copyout>,
    VersionedClause<ACCC_Create>,
    VersionedClause<ACCC_DevicePtr>,
    VersionedClause<ACCC_DeviceType>,
    VersionedClause<ACCC_NoCreate>,
````
- **L341 EN**: Executes a standalone statement or declaration: `VersionedClause<ACCC_Wait>];`.
  **L341 CN**: 执行一条独立语句或声明：`VersionedClause<ACCC_Wait>];`。
- **L342 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L342 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L343 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_If>,`.
  **L343 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_If>,`。
- **L344 EN**: Executes a standalone statement or declaration: `VersionedClause<ACCC_Self>];`.
  **L344 CN**: 执行一条独立语句或声明：`VersionedClause<ACCC_Self>];`。
- **L345 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L345 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L346 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L346 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L347 EN**: Closes the current lexical scope or compound statement.
  **L347 CN**: 结束当前词法作用域或复合语句块。
- **L348 EN**: Blank line separating nearby declarations or logic blocks.
  **L348 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L349 EN**: Comment explains nearby logic, invariants, or intent: `2.5.1`.
  **L349 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.5.1`。
- **L350 EN**: Declares TableGen def `ACC_Parallel`.
  **L350 CN**: 声明 TableGen def `ACC_Parallel`。
- **L351 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L351 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L352 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Attach>,`.
  **L352 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Attach>,`。
- **L353 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Async>,`.
  **L353 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Async>,`。
- **L354 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Copy>,`.
  **L354 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Copy>,`。
- **L355 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Copyin>,`.
  **L355 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Copyin>,`。
- **L356 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Copyout>,`.
  **L356 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Copyout>,`。
- **L357 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Create>,`.
  **L357 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Create>,`。
- **L358 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_DevicePtr>,`.
  **L358 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_DevicePtr>,`。
- **L359 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_DeviceType>,`.
  **L359 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_DeviceType>,`。
- **L360 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_NoCreate>,`.
  **L360 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_NoCreate>,`。

### Lines 361-380

````tablegen
    VersionedClause<ACCC_NumGangs>,
    VersionedClause<ACCC_NumWorkers>,
    VersionedClause<ACCC_Present>,
    VersionedClause<ACCC_Private>,
    VersionedClause<ACCC_FirstPrivate>,
    VersionedClause<ACCC_Reduction>,
    VersionedClause<ACCC_Wait>,
    VersionedClause<ACCC_VectorLength>
  ];
  let allowedOnceClauses = [
    VersionedClause<ACCC_Default>,
    VersionedClause<ACCC_If>,
    VersionedClause<ACCC_Self>
  ];
  let association = AS_Block;
  let category = CA_Executable;
}

// 2.5.2
def ACC_Serial : Directive<[Spelling<"serial">]> {
````
- **L361 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_NumGangs>,`.
  **L361 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_NumGangs>,`。
- **L362 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_NumWorkers>,`.
  **L362 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_NumWorkers>,`。
- **L363 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Present>,`.
  **L363 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Present>,`。
- **L364 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Private>,`.
  **L364 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Private>,`。
- **L365 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_FirstPrivate>,`.
  **L365 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_FirstPrivate>,`。
- **L366 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Reduction>,`.
  **L366 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Reduction>,`。
- **L367 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Wait>,`.
  **L367 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Wait>,`。
- **L368 EN**: Continues the surrounding expression or declaration: `VersionedClause<ACCC_VectorLength>`.
  **L368 CN**: 继续构造周围的表达式或声明：`VersionedClause<ACCC_VectorLength>`。
- **L369 EN**: Executes a standalone statement or declaration: `];`.
  **L369 CN**: 执行一条独立语句或声明：`];`。
- **L370 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L370 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L371 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Default>,`.
  **L371 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Default>,`。
- **L372 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_If>,`.
  **L372 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_If>,`。
- **L373 EN**: Continues the surrounding expression or declaration: `VersionedClause<ACCC_Self>`.
  **L373 CN**: 继续构造周围的表达式或声明：`VersionedClause<ACCC_Self>`。
- **L374 EN**: Executes a standalone statement or declaration: `];`.
  **L374 CN**: 执行一条独立语句或声明：`];`。
- **L375 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L375 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L376 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L376 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L377 EN**: Closes the current lexical scope or compound statement.
  **L377 CN**: 结束当前词法作用域或复合语句块。
- **L378 EN**: Blank line separating nearby declarations or logic blocks.
  **L378 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L379 EN**: Comment explains nearby logic, invariants, or intent: `2.5.2`.
  **L379 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.5.2`。
- **L380 EN**: Declares TableGen def `ACC_Serial`.
  **L380 CN**: 声明 TableGen def `ACC_Serial`。

### Lines 381-400

````tablegen
  // Spec line 950-951: clause is as for the parallel construct except that the
  // num_gangs, num_workers, and vector_length clauses are not permitted.
  let allowedClauses = [VersionedClause<ACCC_Async>,
                        VersionedClause<ACCC_Attach>,
                        VersionedClause<ACCC_Copy>,
                        VersionedClause<ACCC_Copyin>,
                        VersionedClause<ACCC_Copyout>,
                        VersionedClause<ACCC_Create>,
                        VersionedClause<ACCC_DevicePtr>,
                        VersionedClause<ACCC_DeviceType>,
                        VersionedClause<ACCC_NoCreate>,
                        VersionedClause<ACCC_Present>,
                        VersionedClause<ACCC_Private>,
                        VersionedClause<ACCC_FirstPrivate>,
                        VersionedClause<ACCC_Reduction>,
                        VersionedClause<ACCC_Wait>];
  let allowedOnceClauses = [VersionedClause<ACCC_Default>,
                            VersionedClause<ACCC_If>,
                            VersionedClause<ACCC_Self>];
  let association = AS_Block;
````
- **L381 EN**: Comment explains nearby logic, invariants, or intent: `Spec line 950-951: clause is as for the parallel construct except that the`.
  **L381 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Spec line 950-951: clause is as for the parallel construct except that the`。
- **L382 EN**: Comment explains nearby logic, invariants, or intent: `num_gangs, num_workers, and vector_length clauses are not permitted.`.
  **L382 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`num_gangs, num_workers, and vector_length clauses are not permitted.`。
- **L383 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L383 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L384 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Attach>,`.
  **L384 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Attach>,`。
- **L385 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Copy>,`.
  **L385 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Copy>,`。
- **L386 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Copyin>,`.
  **L386 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Copyin>,`。
- **L387 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Copyout>,`.
  **L387 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Copyout>,`。
- **L388 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Create>,`.
  **L388 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Create>,`。
- **L389 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_DevicePtr>,`.
  **L389 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_DevicePtr>,`。
- **L390 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_DeviceType>,`.
  **L390 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_DeviceType>,`。
- **L391 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_NoCreate>,`.
  **L391 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_NoCreate>,`。
- **L392 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Present>,`.
  **L392 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Present>,`。
- **L393 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Private>,`.
  **L393 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Private>,`。
- **L394 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_FirstPrivate>,`.
  **L394 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_FirstPrivate>,`。
- **L395 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Reduction>,`.
  **L395 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Reduction>,`。
- **L396 EN**: Executes a standalone statement or declaration: `VersionedClause<ACCC_Wait>];`.
  **L396 CN**: 执行一条独立语句或声明：`VersionedClause<ACCC_Wait>];`。
- **L397 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L397 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L398 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_If>,`.
  **L398 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_If>,`。
- **L399 EN**: Executes a standalone statement or declaration: `VersionedClause<ACCC_Self>];`.
  **L399 CN**: 执行一条独立语句或声明：`VersionedClause<ACCC_Self>];`。
- **L400 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L400 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 401-420

````tablegen
  let category = CA_Executable;
}

// 2.9
def ACC_Loop : Directive<[Spelling<"loop">]> {
  let allowedClauses = [
    VersionedClause<ACCC_DeviceType>,
    VersionedClause<ACCC_Private>,
    VersionedClause<ACCC_Reduction>,
    VersionedClause<ACCC_Collapse>,
    VersionedClause<ACCC_Gang>,
    VersionedClause<ACCC_ShortLoop>,
    VersionedClause<ACCC_Tile>,
    VersionedClause<ACCC_Vector>,
    VersionedClause<ACCC_Worker>
  ];
  let allowedExclusiveClauses = [
    VersionedClause<ACCC_Auto>,
    VersionedClause<ACCC_Independent>,
    VersionedClause<ACCC_Seq>
````
- **L401 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L401 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L402 EN**: Closes the current lexical scope or compound statement.
  **L402 CN**: 结束当前词法作用域或复合语句块。
- **L403 EN**: Blank line separating nearby declarations or logic blocks.
  **L403 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L404 EN**: Comment explains nearby logic, invariants, or intent: `2.9`.
  **L404 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.9`。
- **L405 EN**: Declares TableGen def `ACC_Loop`.
  **L405 CN**: 声明 TableGen def `ACC_Loop`。
- **L406 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L406 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L407 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_DeviceType>,`.
  **L407 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_DeviceType>,`。
- **L408 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Private>,`.
  **L408 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Private>,`。
- **L409 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Reduction>,`.
  **L409 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Reduction>,`。
- **L410 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Collapse>,`.
  **L410 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Collapse>,`。
- **L411 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Gang>,`.
  **L411 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Gang>,`。
- **L412 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_ShortLoop>,`.
  **L412 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_ShortLoop>,`。
- **L413 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Tile>,`.
  **L413 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Tile>,`。
- **L414 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Vector>,`.
  **L414 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Vector>,`。
- **L415 EN**: Continues the surrounding expression or declaration: `VersionedClause<ACCC_Worker>`.
  **L415 CN**: 继续构造周围的表达式或声明：`VersionedClause<ACCC_Worker>`。
- **L416 EN**: Executes a standalone statement or declaration: `];`.
  **L416 CN**: 执行一条独立语句或声明：`];`。
- **L417 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L417 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L418 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Auto>,`.
  **L418 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Auto>,`。
- **L419 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Independent>,`.
  **L419 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Independent>,`。
- **L420 EN**: Continues the surrounding expression or declaration: `VersionedClause<ACCC_Seq>`.
  **L420 CN**: 继续构造周围的表达式或声明：`VersionedClause<ACCC_Seq>`。

### Lines 421-440

````tablegen
  ];
  let association = AS_LoopNest;
  let category = CA_Executable;
}

// 2.10
def ACC_Cache : Directive<[Spelling<"cache">]> {
  let association = AS_None;
  let category = CA_Executable;
}

// 2.14.1
def ACC_Init : Directive<[Spelling<"init">]> {
  let allowedOnceClauses = [VersionedClause<ACCC_DeviceNum>,
                            VersionedClause<ACCC_If>];
  let allowedClauses = [VersionedClause<ACCC_DeviceType>];
  let association = AS_None;
  let category = CA_Executable;
}

````
- **L421 EN**: Executes a standalone statement or declaration: `];`.
  **L421 CN**: 执行一条独立语句或声明：`];`。
- **L422 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L422 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L423 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L423 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L424 EN**: Closes the current lexical scope or compound statement.
  **L424 CN**: 结束当前词法作用域或复合语句块。
- **L425 EN**: Blank line separating nearby declarations or logic blocks.
  **L425 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L426 EN**: Comment explains nearby logic, invariants, or intent: `2.10`.
  **L426 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.10`。
- **L427 EN**: Declares TableGen def `ACC_Cache`.
  **L427 CN**: 声明 TableGen def `ACC_Cache`。
- **L428 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L428 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L429 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L429 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L430 EN**: Closes the current lexical scope or compound statement.
  **L430 CN**: 结束当前词法作用域或复合语句块。
- **L431 EN**: Blank line separating nearby declarations or logic blocks.
  **L431 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L432 EN**: Comment explains nearby logic, invariants, or intent: `2.14.1`.
  **L432 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.14.1`。
- **L433 EN**: Declares TableGen def `ACC_Init`.
  **L433 CN**: 声明 TableGen def `ACC_Init`。
- **L434 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L434 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L435 EN**: Executes a standalone statement or declaration: `VersionedClause<ACCC_If>];`.
  **L435 CN**: 执行一条独立语句或声明：`VersionedClause<ACCC_If>];`。
- **L436 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L436 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L437 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L437 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L438 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L438 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L439 EN**: Closes the current lexical scope or compound statement.
  **L439 CN**: 结束当前词法作用域或复合语句块。
- **L440 EN**: Blank line separating nearby declarations or logic blocks.
  **L440 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 441-460

````tablegen
// 2.15.1
def ACC_Routine : Directive<[Spelling<"routine">]> {
  let allowedClauses = [
    VersionedClause<ACCC_Bind>,
    VersionedClause<ACCC_DeviceType>,
    VersionedClause<ACCC_Gang>,
    VersionedClause<ACCC_Seq>,
    VersionedClause<ACCC_Vector>,
    VersionedClause<ACCC_Worker>
  ];
  let allowedOnceClauses = [
    VersionedClause<ACCC_NoHost>
  ];
  let association = AS_Declaration;
  let category = CA_Declarative;
}

// 2.14.3
def ACC_Set : Directive<[Spelling<"set">]> {
  let allowedOnceClauses = [
````
- **L441 EN**: Comment explains nearby logic, invariants, or intent: `2.15.1`.
  **L441 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.15.1`。
- **L442 EN**: Declares TableGen def `ACC_Routine`.
  **L442 CN**: 声明 TableGen def `ACC_Routine`。
- **L443 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L443 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L444 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Bind>,`.
  **L444 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Bind>,`。
- **L445 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_DeviceType>,`.
  **L445 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_DeviceType>,`。
- **L446 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Gang>,`.
  **L446 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Gang>,`。
- **L447 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Seq>,`.
  **L447 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Seq>,`。
- **L448 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Vector>,`.
  **L448 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Vector>,`。
- **L449 EN**: Continues the surrounding expression or declaration: `VersionedClause<ACCC_Worker>`.
  **L449 CN**: 继续构造周围的表达式或声明：`VersionedClause<ACCC_Worker>`。
- **L450 EN**: Executes a standalone statement or declaration: `];`.
  **L450 CN**: 执行一条独立语句或声明：`];`。
- **L451 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L451 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L452 EN**: Continues the surrounding expression or declaration: `VersionedClause<ACCC_NoHost>`.
  **L452 CN**: 继续构造周围的表达式或声明：`VersionedClause<ACCC_NoHost>`。
- **L453 EN**: Executes a standalone statement or declaration: `];`.
  **L453 CN**: 执行一条独立语句或声明：`];`。
- **L454 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L454 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L455 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L455 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L456 EN**: Closes the current lexical scope or compound statement.
  **L456 CN**: 结束当前词法作用域或复合语句块。
- **L457 EN**: Blank line separating nearby declarations or logic blocks.
  **L457 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L458 EN**: Comment explains nearby logic, invariants, or intent: `2.14.3`.
  **L458 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.14.3`。
- **L459 EN**: Declares TableGen def `ACC_Set`.
  **L459 CN**: 声明 TableGen def `ACC_Set`。
- **L460 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L460 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 461-480

````tablegen
    VersionedClause<ACCC_DefaultAsync>,
    VersionedClause<ACCC_DeviceNum>,
    VersionedClause<ACCC_DeviceType>,
    VersionedClause<ACCC_If>
  ];
  let requiredClauses = [
    // The three following clauses are also in allowedOnceClauses list due to
    // restriction 2255 - Two instances of the same clause may not appear on the
    // same directive.
    VersionedClause<ACCC_DefaultAsync>,
    VersionedClause<ACCC_DeviceNum>,
    VersionedClause<ACCC_DeviceType>
  ];
  let association = AS_None;
  let category = CA_Executable;
}

// 2.14.2
def ACC_Shutdown : Directive<[Spelling<"shutdown">]> {
  let allowedOnceClauses = [VersionedClause<ACCC_DeviceNum>,
````
- **L461 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_DefaultAsync>,`.
  **L461 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_DefaultAsync>,`。
- **L462 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_DeviceNum>,`.
  **L462 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_DeviceNum>,`。
- **L463 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_DeviceType>,`.
  **L463 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_DeviceType>,`。
- **L464 EN**: Continues the surrounding expression or declaration: `VersionedClause<ACCC_If>`.
  **L464 CN**: 继续构造周围的表达式或声明：`VersionedClause<ACCC_If>`。
- **L465 EN**: Executes a standalone statement or declaration: `];`.
  **L465 CN**: 执行一条独立语句或声明：`];`。
- **L466 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L466 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L467 EN**: Comment explains nearby logic, invariants, or intent: `The three following clauses are also in allowedOnceClauses list due to`.
  **L467 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The three following clauses are also in allowedOnceClauses list due to`。
- **L468 EN**: Comment explains nearby logic, invariants, or intent: `restriction 2255 - Two instances of the same clause may not appear on the`.
  **L468 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`restriction 2255 - Two instances of the same clause may not appear on the`。
- **L469 EN**: Comment explains nearby logic, invariants, or intent: `same directive.`.
  **L469 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`same directive.`。
- **L470 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_DefaultAsync>,`.
  **L470 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_DefaultAsync>,`。
- **L471 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_DeviceNum>,`.
  **L471 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_DeviceNum>,`。
- **L472 EN**: Continues the surrounding expression or declaration: `VersionedClause<ACCC_DeviceType>`.
  **L472 CN**: 继续构造周围的表达式或声明：`VersionedClause<ACCC_DeviceType>`。
- **L473 EN**: Executes a standalone statement or declaration: `];`.
  **L473 CN**: 执行一条独立语句或声明：`];`。
- **L474 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L474 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L475 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L475 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L476 EN**: Closes the current lexical scope or compound statement.
  **L476 CN**: 结束当前词法作用域或复合语句块。
- **L477 EN**: Blank line separating nearby declarations or logic blocks.
  **L477 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L478 EN**: Comment explains nearby logic, invariants, or intent: `2.14.2`.
  **L478 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.14.2`。
- **L479 EN**: Declares TableGen def `ACC_Shutdown`.
  **L479 CN**: 声明 TableGen def `ACC_Shutdown`。
- **L480 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L480 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 481-500

````tablegen
                            VersionedClause<ACCC_If>];
  let allowedClauses = [VersionedClause<ACCC_DeviceType>];
  let association = AS_None;
  let category = CA_Executable;
}

// 2.14.4
def ACC_Update : Directive<[Spelling<"update">]> {
  let allowedClauses = [VersionedClause<ACCC_DeviceType>,
                        VersionedClause<ACCC_IfPresent>,
                        VersionedClause<ACCC_Wait>];
  let allowedOnceClauses = [VersionedClause<ACCC_Async>,
                            VersionedClause<ACCC_If>];
  let requiredClauses = [
    VersionedClause<ACCC_Device>,
    VersionedClause<ACCC_Host>,
    VersionedClause<ACCC_Self>
  ];
  let association = AS_None;
  let category = CA_Executable;
````
- **L481 EN**: Executes a standalone statement or declaration: `VersionedClause<ACCC_If>];`.
  **L481 CN**: 执行一条独立语句或声明：`VersionedClause<ACCC_If>];`。
- **L482 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L482 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L483 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L483 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L484 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L484 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L485 EN**: Closes the current lexical scope or compound statement.
  **L485 CN**: 结束当前词法作用域或复合语句块。
- **L486 EN**: Blank line separating nearby declarations or logic blocks.
  **L486 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L487 EN**: Comment explains nearby logic, invariants, or intent: `2.14.4`.
  **L487 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.14.4`。
- **L488 EN**: Declares TableGen def `ACC_Update`.
  **L488 CN**: 声明 TableGen def `ACC_Update`。
- **L489 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L489 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L490 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_IfPresent>,`.
  **L490 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_IfPresent>,`。
- **L491 EN**: Executes a standalone statement or declaration: `VersionedClause<ACCC_Wait>];`.
  **L491 CN**: 执行一条独立语句或声明：`VersionedClause<ACCC_Wait>];`。
- **L492 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L492 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L493 EN**: Executes a standalone statement or declaration: `VersionedClause<ACCC_If>];`.
  **L493 CN**: 执行一条独立语句或声明：`VersionedClause<ACCC_If>];`。
- **L494 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L494 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L495 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Device>,`.
  **L495 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Device>,`。
- **L496 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Host>,`.
  **L496 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Host>,`。
- **L497 EN**: Continues the surrounding expression or declaration: `VersionedClause<ACCC_Self>`.
  **L497 CN**: 继续构造周围的表达式或声明：`VersionedClause<ACCC_Self>`。
- **L498 EN**: Executes a standalone statement or declaration: `];`.
  **L498 CN**: 执行一条独立语句或声明：`];`。
- **L499 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L499 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L500 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L500 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 501-520

````tablegen
}

// 2.16.3
def ACC_Wait : Directive<[Spelling<"wait">]> {
  let allowedOnceClauses = [
    VersionedClause<ACCC_Async>,
    VersionedClause<ACCC_If>
  ];
  let association = AS_None;
  let category = CA_Executable;
}

// 2.14.6
def ACC_EnterData : Directive<[Spelling<"enter data">]> {
  let allowedClauses = [
    VersionedClause<ACCC_Wait>
  ];
  let allowedOnceClauses = [
    VersionedClause<ACCC_Async>,
    VersionedClause<ACCC_If>
````
- **L501 EN**: Closes the current lexical scope or compound statement.
  **L501 CN**: 结束当前词法作用域或复合语句块。
- **L502 EN**: Blank line separating nearby declarations or logic blocks.
  **L502 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L503 EN**: Comment explains nearby logic, invariants, or intent: `2.16.3`.
  **L503 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.16.3`。
- **L504 EN**: Declares TableGen def `ACC_Wait`.
  **L504 CN**: 声明 TableGen def `ACC_Wait`。
- **L505 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L505 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L506 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Async>,`.
  **L506 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Async>,`。
- **L507 EN**: Continues the surrounding expression or declaration: `VersionedClause<ACCC_If>`.
  **L507 CN**: 继续构造周围的表达式或声明：`VersionedClause<ACCC_If>`。
- **L508 EN**: Executes a standalone statement or declaration: `];`.
  **L508 CN**: 执行一条独立语句或声明：`];`。
- **L509 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L509 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L510 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L510 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L511 EN**: Closes the current lexical scope or compound statement.
  **L511 CN**: 结束当前词法作用域或复合语句块。
- **L512 EN**: Blank line separating nearby declarations or logic blocks.
  **L512 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L513 EN**: Comment explains nearby logic, invariants, or intent: `2.14.6`.
  **L513 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.14.6`。
- **L514 EN**: Declares TableGen def `ACC_EnterData`.
  **L514 CN**: 声明 TableGen def `ACC_EnterData`。
- **L515 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L515 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L516 EN**: Continues the surrounding expression or declaration: `VersionedClause<ACCC_Wait>`.
  **L516 CN**: 继续构造周围的表达式或声明：`VersionedClause<ACCC_Wait>`。
- **L517 EN**: Executes a standalone statement or declaration: `];`.
  **L517 CN**: 执行一条独立语句或声明：`];`。
- **L518 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L518 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L519 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Async>,`.
  **L519 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Async>,`。
- **L520 EN**: Continues the surrounding expression or declaration: `VersionedClause<ACCC_If>`.
  **L520 CN**: 继续构造周围的表达式或声明：`VersionedClause<ACCC_If>`。

### Lines 521-540

````tablegen
  ];
  let requiredClauses = [
    VersionedClause<ACCC_Attach>,
    VersionedClause<ACCC_Create>,
    VersionedClause<ACCC_Copyin>
  ];
  let association = AS_None;
  let category = CA_Executable;
}

// 2.14.7
def ACC_ExitData : Directive<[Spelling<"exit data">]> {
  let allowedClauses = [VersionedClause<ACCC_Finalize>,
                        VersionedClause<ACCC_Wait>];
  let allowedOnceClauses = [VersionedClause<ACCC_Async>,
                            VersionedClause<ACCC_If>];
  let requiredClauses = [
    VersionedClause<ACCC_Copyout>,
    VersionedClause<ACCC_Delete>,
    VersionedClause<ACCC_Detach>
````
- **L521 EN**: Executes a standalone statement or declaration: `];`.
  **L521 CN**: 执行一条独立语句或声明：`];`。
- **L522 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L522 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L523 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Attach>,`.
  **L523 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Attach>,`。
- **L524 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Create>,`.
  **L524 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Create>,`。
- **L525 EN**: Continues the surrounding expression or declaration: `VersionedClause<ACCC_Copyin>`.
  **L525 CN**: 继续构造周围的表达式或声明：`VersionedClause<ACCC_Copyin>`。
- **L526 EN**: Executes a standalone statement or declaration: `];`.
  **L526 CN**: 执行一条独立语句或声明：`];`。
- **L527 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L527 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L528 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L528 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L529 EN**: Closes the current lexical scope or compound statement.
  **L529 CN**: 结束当前词法作用域或复合语句块。
- **L530 EN**: Blank line separating nearby declarations or logic blocks.
  **L530 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L531 EN**: Comment explains nearby logic, invariants, or intent: `2.14.7`.
  **L531 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.14.7`。
- **L532 EN**: Declares TableGen def `ACC_ExitData`.
  **L532 CN**: 声明 TableGen def `ACC_ExitData`。
- **L533 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L533 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L534 EN**: Executes a standalone statement or declaration: `VersionedClause<ACCC_Wait>];`.
  **L534 CN**: 执行一条独立语句或声明：`VersionedClause<ACCC_Wait>];`。
- **L535 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L535 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L536 EN**: Executes a standalone statement or declaration: `VersionedClause<ACCC_If>];`.
  **L536 CN**: 执行一条独立语句或声明：`VersionedClause<ACCC_If>];`。
- **L537 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L537 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L538 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Copyout>,`.
  **L538 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Copyout>,`。
- **L539 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Delete>,`.
  **L539 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Delete>,`。
- **L540 EN**: Continues the surrounding expression or declaration: `VersionedClause<ACCC_Detach>`.
  **L540 CN**: 继续构造周围的表达式或声明：`VersionedClause<ACCC_Detach>`。

### Lines 541-560

````tablegen
  ];
  let association = AS_None;
  let category = CA_Executable;
}

// 2.8
def ACC_HostData : Directive<[Spelling<"host_data">]> {
  let allowedClauses = [VersionedClause<ACCC_IfPresent>];
  let allowedOnceClauses = [VersionedClause<ACCC_If>];
  let requiredClauses = [
    VersionedClause<ACCC_UseDevice>
  ];
  let association = AS_Block;
  let category = CA_Executable;
}

// 2.11
def ACC_KernelsLoop : Directive<[Spelling<"kernels loop">]> {
  let allowedClauses = [VersionedClause<ACCC_Async>,
                        VersionedClause<ACCC_Attach>,
````
- **L541 EN**: Executes a standalone statement or declaration: `];`.
  **L541 CN**: 执行一条独立语句或声明：`];`。
- **L542 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L542 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L543 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L543 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L544 EN**: Closes the current lexical scope or compound statement.
  **L544 CN**: 结束当前词法作用域或复合语句块。
- **L545 EN**: Blank line separating nearby declarations or logic blocks.
  **L545 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L546 EN**: Comment explains nearby logic, invariants, or intent: `2.8`.
  **L546 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.8`。
- **L547 EN**: Declares TableGen def `ACC_HostData`.
  **L547 CN**: 声明 TableGen def `ACC_HostData`。
- **L548 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L548 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L549 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L549 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L550 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L550 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L551 EN**: Continues the surrounding expression or declaration: `VersionedClause<ACCC_UseDevice>`.
  **L551 CN**: 继续构造周围的表达式或声明：`VersionedClause<ACCC_UseDevice>`。
- **L552 EN**: Executes a standalone statement or declaration: `];`.
  **L552 CN**: 执行一条独立语句或声明：`];`。
- **L553 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L553 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L554 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L554 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L555 EN**: Closes the current lexical scope or compound statement.
  **L555 CN**: 结束当前词法作用域或复合语句块。
- **L556 EN**: Blank line separating nearby declarations or logic blocks.
  **L556 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L557 EN**: Comment explains nearby logic, invariants, or intent: `2.11`.
  **L557 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.11`。
- **L558 EN**: Declares TableGen def `ACC_KernelsLoop`.
  **L558 CN**: 声明 TableGen def `ACC_KernelsLoop`。
- **L559 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L559 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L560 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Attach>,`.
  **L560 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Attach>,`。

### Lines 561-580

````tablegen
                        VersionedClause<ACCC_Collapse>,
                        VersionedClause<ACCC_Copy>,
                        VersionedClause<ACCC_Copyin>,
                        VersionedClause<ACCC_Copyout>,
                        VersionedClause<ACCC_Create>,
                        VersionedClause<ACCC_DevicePtr>,
                        VersionedClause<ACCC_DeviceType>,
                        VersionedClause<ACCC_Gang>,
                        VersionedClause<ACCC_NoCreate>,
                        VersionedClause<ACCC_NumGangs>,
                        VersionedClause<ACCC_NumWorkers>,
                        VersionedClause<ACCC_Present>,
                        VersionedClause<ACCC_Private>,
                        VersionedClause<ACCC_Reduction>,
                        VersionedClause<ACCC_ShortLoop>,
                        VersionedClause<ACCC_Tile>,
                        VersionedClause<ACCC_Vector>,
                        VersionedClause<ACCC_VectorLength>,
                        VersionedClause<ACCC_Wait>,
                        VersionedClause<ACCC_Worker>];
````
- **L561 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Collapse>,`.
  **L561 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Collapse>,`。
- **L562 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Copy>,`.
  **L562 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Copy>,`。
- **L563 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Copyin>,`.
  **L563 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Copyin>,`。
- **L564 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Copyout>,`.
  **L564 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Copyout>,`。
- **L565 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Create>,`.
  **L565 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Create>,`。
- **L566 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_DevicePtr>,`.
  **L566 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_DevicePtr>,`。
- **L567 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_DeviceType>,`.
  **L567 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_DeviceType>,`。
- **L568 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Gang>,`.
  **L568 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Gang>,`。
- **L569 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_NoCreate>,`.
  **L569 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_NoCreate>,`。
- **L570 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_NumGangs>,`.
  **L570 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_NumGangs>,`。
- **L571 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_NumWorkers>,`.
  **L571 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_NumWorkers>,`。
- **L572 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Present>,`.
  **L572 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Present>,`。
- **L573 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Private>,`.
  **L573 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Private>,`。
- **L574 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Reduction>,`.
  **L574 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Reduction>,`。
- **L575 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_ShortLoop>,`.
  **L575 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_ShortLoop>,`。
- **L576 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Tile>,`.
  **L576 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Tile>,`。
- **L577 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Vector>,`.
  **L577 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Vector>,`。
- **L578 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_VectorLength>,`.
  **L578 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_VectorLength>,`。
- **L579 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Wait>,`.
  **L579 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Wait>,`。
- **L580 EN**: Executes a standalone statement or declaration: `VersionedClause<ACCC_Worker>];`.
  **L580 CN**: 执行一条独立语句或声明：`VersionedClause<ACCC_Worker>];`。

### Lines 581-600

````tablegen
  let allowedOnceClauses = [VersionedClause<ACCC_Default>,
                            VersionedClause<ACCC_If>,
                            VersionedClause<ACCC_Self>];
  let allowedExclusiveClauses = [
    VersionedClause<ACCC_Auto>,
    VersionedClause<ACCC_Independent>,
    VersionedClause<ACCC_Seq>
  ];
  let leafConstructs = [ACC_Kernels, ACC_Loop];
  let category = CA_Executable;
}

// 2.11
def ACC_ParallelLoop : Directive<[Spelling<"parallel loop">]> {
  let allowedClauses = [VersionedClause<ACCC_Async>,
                        VersionedClause<ACCC_Attach>,
                        VersionedClause<ACCC_Collapse>,
                        VersionedClause<ACCC_Copy>,
                        VersionedClause<ACCC_Copyin>,
                        VersionedClause<ACCC_Copyout>,
````
- **L581 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L581 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L582 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_If>,`.
  **L582 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_If>,`。
- **L583 EN**: Executes a standalone statement or declaration: `VersionedClause<ACCC_Self>];`.
  **L583 CN**: 执行一条独立语句或声明：`VersionedClause<ACCC_Self>];`。
- **L584 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L584 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L585 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Auto>,`.
  **L585 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Auto>,`。
- **L586 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Independent>,`.
  **L586 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Independent>,`。
- **L587 EN**: Continues the surrounding expression or declaration: `VersionedClause<ACCC_Seq>`.
  **L587 CN**: 继续构造周围的表达式或声明：`VersionedClause<ACCC_Seq>`。
- **L588 EN**: Executes a standalone statement or declaration: `];`.
  **L588 CN**: 执行一条独立语句或声明：`];`。
- **L589 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L589 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L590 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L590 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L591 EN**: Closes the current lexical scope or compound statement.
  **L591 CN**: 结束当前词法作用域或复合语句块。
- **L592 EN**: Blank line separating nearby declarations or logic blocks.
  **L592 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L593 EN**: Comment explains nearby logic, invariants, or intent: `2.11`.
  **L593 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.11`。
- **L594 EN**: Declares TableGen def `ACC_ParallelLoop`.
  **L594 CN**: 声明 TableGen def `ACC_ParallelLoop`。
- **L595 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L595 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L596 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Attach>,`.
  **L596 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Attach>,`。
- **L597 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Collapse>,`.
  **L597 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Collapse>,`。
- **L598 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Copy>,`.
  **L598 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Copy>,`。
- **L599 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Copyin>,`.
  **L599 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Copyin>,`。
- **L600 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Copyout>,`.
  **L600 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Copyout>,`。

### Lines 601-620

````tablegen
                        VersionedClause<ACCC_Create>,
                        VersionedClause<ACCC_DevicePtr>,
                        VersionedClause<ACCC_DeviceType>,
                        VersionedClause<ACCC_FirstPrivate>,
                        VersionedClause<ACCC_Gang>,
                        VersionedClause<ACCC_NoCreate>,
                        VersionedClause<ACCC_NumGangs>,
                        VersionedClause<ACCC_NumWorkers>,
                        VersionedClause<ACCC_Present>,
                        VersionedClause<ACCC_Private>,
                        VersionedClause<ACCC_Reduction>,
                        VersionedClause<ACCC_ShortLoop>,
                        VersionedClause<ACCC_Tile>,
                        VersionedClause<ACCC_Vector>,
                        VersionedClause<ACCC_VectorLength>,
                        VersionedClause<ACCC_Wait>,
                        VersionedClause<ACCC_Worker>];
  let allowedOnceClauses = [VersionedClause<ACCC_Default>,
                            VersionedClause<ACCC_If>,
                            VersionedClause<ACCC_Self>];
````
- **L601 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Create>,`.
  **L601 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Create>,`。
- **L602 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_DevicePtr>,`.
  **L602 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_DevicePtr>,`。
- **L603 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_DeviceType>,`.
  **L603 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_DeviceType>,`。
- **L604 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_FirstPrivate>,`.
  **L604 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_FirstPrivate>,`。
- **L605 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Gang>,`.
  **L605 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Gang>,`。
- **L606 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_NoCreate>,`.
  **L606 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_NoCreate>,`。
- **L607 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_NumGangs>,`.
  **L607 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_NumGangs>,`。
- **L608 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_NumWorkers>,`.
  **L608 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_NumWorkers>,`。
- **L609 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Present>,`.
  **L609 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Present>,`。
- **L610 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Private>,`.
  **L610 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Private>,`。
- **L611 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Reduction>,`.
  **L611 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Reduction>,`。
- **L612 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_ShortLoop>,`.
  **L612 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_ShortLoop>,`。
- **L613 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Tile>,`.
  **L613 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Tile>,`。
- **L614 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Vector>,`.
  **L614 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Vector>,`。
- **L615 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_VectorLength>,`.
  **L615 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_VectorLength>,`。
- **L616 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Wait>,`.
  **L616 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Wait>,`。
- **L617 EN**: Executes a standalone statement or declaration: `VersionedClause<ACCC_Worker>];`.
  **L617 CN**: 执行一条独立语句或声明：`VersionedClause<ACCC_Worker>];`。
- **L618 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L618 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L619 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_If>,`.
  **L619 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_If>,`。
- **L620 EN**: Executes a standalone statement or declaration: `VersionedClause<ACCC_Self>];`.
  **L620 CN**: 执行一条独立语句或声明：`VersionedClause<ACCC_Self>];`。

### Lines 621-640

````tablegen
  let allowedExclusiveClauses = [
    VersionedClause<ACCC_Auto>,
    VersionedClause<ACCC_Independent>,
    VersionedClause<ACCC_Seq>
  ];
  let leafConstructs = [ACC_Parallel, ACC_Loop];
  let category = CA_Executable;
}

// 2.11
def ACC_SerialLoop : Directive<[Spelling<"serial loop">]> {
  let allowedClauses = [VersionedClause<ACCC_Async>,
                        VersionedClause<ACCC_Attach>,
                        VersionedClause<ACCC_Collapse>,
                        VersionedClause<ACCC_Copy>,
                        VersionedClause<ACCC_Copyin>,
                        VersionedClause<ACCC_Copyout>,
                        VersionedClause<ACCC_Create>,
                        VersionedClause<ACCC_DevicePtr>,
                        VersionedClause<ACCC_DeviceType>,
````
- **L621 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L621 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L622 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Auto>,`.
  **L622 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Auto>,`。
- **L623 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Independent>,`.
  **L623 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Independent>,`。
- **L624 EN**: Continues the surrounding expression or declaration: `VersionedClause<ACCC_Seq>`.
  **L624 CN**: 继续构造周围的表达式或声明：`VersionedClause<ACCC_Seq>`。
- **L625 EN**: Executes a standalone statement or declaration: `];`.
  **L625 CN**: 执行一条独立语句或声明：`];`。
- **L626 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L626 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L627 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L627 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L628 EN**: Closes the current lexical scope or compound statement.
  **L628 CN**: 结束当前词法作用域或复合语句块。
- **L629 EN**: Blank line separating nearby declarations or logic blocks.
  **L629 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L630 EN**: Comment explains nearby logic, invariants, or intent: `2.11`.
  **L630 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2.11`。
- **L631 EN**: Declares TableGen def `ACC_SerialLoop`.
  **L631 CN**: 声明 TableGen def `ACC_SerialLoop`。
- **L632 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L632 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L633 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Attach>,`.
  **L633 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Attach>,`。
- **L634 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Collapse>,`.
  **L634 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Collapse>,`。
- **L635 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Copy>,`.
  **L635 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Copy>,`。
- **L636 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Copyin>,`.
  **L636 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Copyin>,`。
- **L637 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Copyout>,`.
  **L637 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Copyout>,`。
- **L638 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Create>,`.
  **L638 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Create>,`。
- **L639 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_DevicePtr>,`.
  **L639 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_DevicePtr>,`。
- **L640 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_DeviceType>,`.
  **L640 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_DeviceType>,`。

### Lines 641-660

````tablegen
                        VersionedClause<ACCC_FirstPrivate>,
                        VersionedClause<ACCC_Gang>,
                        VersionedClause<ACCC_NoCreate>,
                        VersionedClause<ACCC_Present>,
                        VersionedClause<ACCC_Private>,
                        VersionedClause<ACCC_Reduction>,
                        VersionedClause<ACCC_ShortLoop>,
                        VersionedClause<ACCC_Tile>,
                        VersionedClause<ACCC_Vector>,
                        VersionedClause<ACCC_Wait>,
                        VersionedClause<ACCC_Worker>];
  let allowedOnceClauses = [VersionedClause<ACCC_Default>,
                            VersionedClause<ACCC_If>,
                            VersionedClause<ACCC_Self>];
  let allowedExclusiveClauses = [
    VersionedClause<ACCC_Auto>,
    VersionedClause<ACCC_Independent>,
    VersionedClause<ACCC_Seq>
  ];
  let leafConstructs = [ACC_Serial, ACC_Loop];
````
- **L641 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_FirstPrivate>,`.
  **L641 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_FirstPrivate>,`。
- **L642 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Gang>,`.
  **L642 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Gang>,`。
- **L643 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_NoCreate>,`.
  **L643 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_NoCreate>,`。
- **L644 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Present>,`.
  **L644 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Present>,`。
- **L645 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Private>,`.
  **L645 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Private>,`。
- **L646 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Reduction>,`.
  **L646 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Reduction>,`。
- **L647 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_ShortLoop>,`.
  **L647 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_ShortLoop>,`。
- **L648 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Tile>,`.
  **L648 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Tile>,`。
- **L649 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Vector>,`.
  **L649 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Vector>,`。
- **L650 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Wait>,`.
  **L650 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Wait>,`。
- **L651 EN**: Executes a standalone statement or declaration: `VersionedClause<ACCC_Worker>];`.
  **L651 CN**: 执行一条独立语句或声明：`VersionedClause<ACCC_Worker>];`。
- **L652 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L652 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L653 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_If>,`.
  **L653 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_If>,`。
- **L654 EN**: Executes a standalone statement or declaration: `VersionedClause<ACCC_Self>];`.
  **L654 CN**: 执行一条独立语句或声明：`VersionedClause<ACCC_Self>];`。
- **L655 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L655 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L656 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Auto>,`.
  **L656 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Auto>,`。
- **L657 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<ACCC_Independent>,`.
  **L657 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<ACCC_Independent>,`。
- **L658 EN**: Continues the surrounding expression or declaration: `VersionedClause<ACCC_Seq>`.
  **L658 CN**: 继续构造周围的表达式或声明：`VersionedClause<ACCC_Seq>`。
- **L659 EN**: Executes a standalone statement or declaration: `];`.
  **L659 CN**: 执行一条独立语句或声明：`];`。
- **L660 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L660 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 661-668

````tablegen
  let category = CA_Executable;
}

def ACC_Unknown : Directive<[Spelling<"unknown">]> {
  let isDefault = true;
  let association = AS_None;
  let category = CA_Utility;
}
````
- **L661 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L661 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L662 EN**: Closes the current lexical scope or compound statement.
  **L662 CN**: 结束当前词法作用域或复合语句块。
- **L663 EN**: Blank line separating nearby declarations or logic blocks.
  **L663 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L664 EN**: Declares TableGen def `ACC_Unknown`.
  **L664 CN**: 声明 TableGen def `ACC_Unknown`。
- **L665 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L665 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L666 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L666 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L667 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L667 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L668 EN**: Closes the current lexical scope or compound statement.
  **L668 CN**: 结束当前词法作用域或复合语句块。

## Key Concepts / 关键概念

- **TableGen record modeling / TableGen 记录建模**

## Dependencies / 依赖关系

- `llvm/Frontend/Directive/DirectiveBase.td`: Provides frontend-facing integration helpers. / 提供面向前端的集成辅助组件。
