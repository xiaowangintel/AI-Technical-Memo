# OMP.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/include/llvm/Frontend/OpenMP/OMP.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: This is the definition file for OpenMP directives and clauses.
- **Purpose (CN)**: 该 TableGen 文件位于 `llvm/include/llvm/Frontend/OpenMP`，主要定义与 `OMP` 相关的记录、生成元数据和声明式规则。

## Line-by-Line Analysis / 逐行分析

### Lines 1-32

````tablegen
//===-- OMP.td - OpenMP directive definition file ----------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This is the definition file for OpenMP directives and clauses.
//
//===----------------------------------------------------------------------===//

include "llvm/Frontend/Directive/DirectiveBase.td"

//===----------------------------------------------------------------------===//
// Definition of general OpenMP information
//===----------------------------------------------------------------------===//

def OpenMP : DirectiveLanguage {
  let name = "OpenMP";
  let cppNamespace = "omp"; // final namespace will be llvm::omp
  let directivePrefix = "OMPD_";
  let clausePrefix = "OMPC_";
  let makeEnumAvailableInNamespace = true;
  let enableBitmaskEnumInNamespace = true;
  let clauseEnumSetClass = "OmpClauseSet";
  let flangClauseBaseClass = "OmpClause";
}

//===----------------------------------------------------------------------===//
// Definitions of OpenMP clauses
// Sorted alphabetically wrt clause spelling.
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
- **L9 EN**: Comment explains nearby logic, invariants, or intent: `This is the definition file for OpenMP directives and clauses.`.
  **L9 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This is the definition file for OpenMP directives and clauses.`。
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
- **L16 EN**: Comment explains nearby logic, invariants, or intent: `Definition of general OpenMP information`.
  **L16 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Definition of general OpenMP information`。
- **L17 EN**: Banner comment marking a file or section boundary.
  **L17 CN**: 横幅注释，用于标记文件或章节边界。
- **L18 EN**: Blank line separating nearby declarations or logic blocks.
  **L18 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L19 EN**: Declares TableGen def `OpenMP`.
  **L19 CN**: 声明 TableGen def `OpenMP`。
- **L20 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L20 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
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
- **L31 EN**: Comment explains nearby logic, invariants, or intent: `Definitions of OpenMP clauses`.
  **L31 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Definitions of OpenMP clauses`。
- **L32 EN**: Comment explains nearby logic, invariants, or intent: `Sorted alphabetically wrt clause spelling.`.
  **L32 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Sorted alphabetically wrt clause spelling.`。

### Lines 33-64

````tablegen
//===----------------------------------------------------------------------===//

def OMPC_Absent : Clause<[Spelling<"absent">]> {
  let clangClass = "OMPAbsentClause";
  let flangClass = "OmpAbsentClause";
}
def OMPC_Acquire : Clause<[Spelling<"acquire">]> {
  let clangClass = "OMPAcquireClause";
}
def OMPC_AcqRel : Clause<[Spelling<"acq_rel">]> {
  let clangClass = "OMPAcqRelClause";
}
def OMPC_AdjustArgs : Clause<[Spelling<"adjust_args">]> {
  let flangClass = "OmpAdjustArgsClause";
}
def OMPC_Affinity : Clause<[Spelling<"affinity">]> {
  let clangClass = "OMPAffinityClause";
  let flangClass = "OmpAffinityClause";
}
def OMPC_Align : Clause<[Spelling<"align">]> {
  let clangClass = "OMPAlignClause";
  let flangClass = "OmpAlignClause";
}
def OMPC_Aligned : Clause<[Spelling<"aligned">]> {
  let clangClass = "OMPAlignedClause";
  let flangClass = "OmpAlignedClause";
}
def OMPC_Allocate : Clause<[Spelling<"allocate">]> {
  let clangClass = "OMPAllocateClause";
  let flangClass = "OmpAllocateClause";
}
def OMPC_Allocator : Clause<[Spelling<"allocator">]> {
````
- **L33 EN**: Banner comment marking a file or section boundary.
  **L33 CN**: 横幅注释，用于标记文件或章节边界。
- **L34 EN**: Blank line separating nearby declarations or logic blocks.
  **L34 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L35 EN**: Declares TableGen def `OMPC_Absent`.
  **L35 CN**: 声明 TableGen def `OMPC_Absent`。
- **L36 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L36 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L37 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L37 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L38 EN**: Closes the current lexical scope or compound statement.
  **L38 CN**: 结束当前词法作用域或复合语句块。
- **L39 EN**: Declares TableGen def `OMPC_Acquire`.
  **L39 CN**: 声明 TableGen def `OMPC_Acquire`。
- **L40 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L40 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L41 EN**: Closes the current lexical scope or compound statement.
  **L41 CN**: 结束当前词法作用域或复合语句块。
- **L42 EN**: Declares TableGen def `OMPC_AcqRel`.
  **L42 CN**: 声明 TableGen def `OMPC_AcqRel`。
- **L43 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L43 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L44 EN**: Closes the current lexical scope or compound statement.
  **L44 CN**: 结束当前词法作用域或复合语句块。
- **L45 EN**: Declares TableGen def `OMPC_AdjustArgs`.
  **L45 CN**: 声明 TableGen def `OMPC_AdjustArgs`。
- **L46 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L46 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L47 EN**: Closes the current lexical scope or compound statement.
  **L47 CN**: 结束当前词法作用域或复合语句块。
- **L48 EN**: Declares TableGen def `OMPC_Affinity`.
  **L48 CN**: 声明 TableGen def `OMPC_Affinity`。
- **L49 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L49 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L50 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L50 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L51 EN**: Closes the current lexical scope or compound statement.
  **L51 CN**: 结束当前词法作用域或复合语句块。
- **L52 EN**: Declares TableGen def `OMPC_Align`.
  **L52 CN**: 声明 TableGen def `OMPC_Align`。
- **L53 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L53 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L54 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L54 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L55 EN**: Closes the current lexical scope or compound statement.
  **L55 CN**: 结束当前词法作用域或复合语句块。
- **L56 EN**: Declares TableGen def `OMPC_Aligned`.
  **L56 CN**: 声明 TableGen def `OMPC_Aligned`。
- **L57 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L57 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L58 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L58 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L59 EN**: Closes the current lexical scope or compound statement.
  **L59 CN**: 结束当前词法作用域或复合语句块。
- **L60 EN**: Declares TableGen def `OMPC_Allocate`.
  **L60 CN**: 声明 TableGen def `OMPC_Allocate`。
- **L61 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L61 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L62 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L62 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L63 EN**: Closes the current lexical scope or compound statement.
  **L63 CN**: 结束当前词法作用域或复合语句块。
- **L64 EN**: Declares TableGen def `OMPC_Allocator`.
  **L64 CN**: 声明 TableGen def `OMPC_Allocator`。

### Lines 65-96

````tablegen
  let clangClass = "OMPAllocatorClause";
  let flangClass = "ScalarIntExpr";
}
def OMPC_Apply : Clause<[Spelling<"apply">]> {
}
def OMPC_AppendArgs : Clause<[Spelling<"append_args">]> {
  let flangClass = "OmpAppendArgsClause";
}
def OMPC_At : Clause<[Spelling<"at">]> {
  let clangClass = "OMPAtClause";
  let flangClass = "OmpAtClause";
}
def OMPC_AtomicDefaultMemOrder
    : Clause<[Spelling<"atomic_default_mem_order">]> {
  let clangClass = "OMPAtomicDefaultMemOrderClause";
  let flangClass = "OmpAtomicDefaultMemOrderClause";
}

def OMP_BIND_parallel : EnumVal<"parallel",1,1> {}
def OMP_BIND_teams : EnumVal<"teams",2,1> {}
def OMP_BIND_thread : EnumVal<"thread",3,1> { let isDefault = true; }
def OMPC_Bind : Clause<[Spelling<"bind">]> {
  let clangClass = "OMPBindClause";
  let flangClass = "OmpBindClause";
  let enumClauseValue = "BindKind";
  let allowedClauseValues = [
    OMP_BIND_parallel,
    OMP_BIND_teams,
    OMP_BIND_thread
  ];
}

````
- **L65 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L65 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L66 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L66 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L67 EN**: Closes the current lexical scope or compound statement.
  **L67 CN**: 结束当前词法作用域或复合语句块。
- **L68 EN**: Declares TableGen def `OMPC_Apply`.
  **L68 CN**: 声明 TableGen def `OMPC_Apply`。
- **L69 EN**: Closes the current lexical scope or compound statement.
  **L69 CN**: 结束当前词法作用域或复合语句块。
- **L70 EN**: Declares TableGen def `OMPC_AppendArgs`.
  **L70 CN**: 声明 TableGen def `OMPC_AppendArgs`。
- **L71 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L71 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L72 EN**: Closes the current lexical scope or compound statement.
  **L72 CN**: 结束当前词法作用域或复合语句块。
- **L73 EN**: Declares TableGen def `OMPC_At`.
  **L73 CN**: 声明 TableGen def `OMPC_At`。
- **L74 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L74 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L75 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L75 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L76 EN**: Closes the current lexical scope or compound statement.
  **L76 CN**: 结束当前词法作用域或复合语句块。
- **L77 EN**: Declares TableGen def `OMPC_AtomicDefaultMemOrder`.
  **L77 CN**: 声明 TableGen def `OMPC_AtomicDefaultMemOrder`。
- **L78 EN**: Continues the surrounding expression or declaration: `: Clause<[Spelling<"atomic_default_mem_order">]> {`.
  **L78 CN**: 继续构造周围的表达式或声明：`: Clause<[Spelling<"atomic_default_mem_order">]> {`。
- **L79 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L79 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L80 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L80 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L81 EN**: Closes the current lexical scope or compound statement.
  **L81 CN**: 结束当前词法作用域或复合语句块。
- **L82 EN**: Blank line separating nearby declarations or logic blocks.
  **L82 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L83 EN**: Declares TableGen def `OMP_BIND_parallel`.
  **L83 CN**: 声明 TableGen def `OMP_BIND_parallel`。
- **L84 EN**: Declares TableGen def `OMP_BIND_teams`.
  **L84 CN**: 声明 TableGen def `OMP_BIND_teams`。
- **L85 EN**: Declares TableGen def `OMP_BIND_thread`.
  **L85 CN**: 声明 TableGen def `OMP_BIND_thread`。
- **L86 EN**: Declares TableGen def `OMPC_Bind`.
  **L86 CN**: 声明 TableGen def `OMPC_Bind`。
- **L87 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L87 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L88 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L88 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L89 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L89 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L90 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L90 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L91 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OMP_BIND_parallel,`.
  **L91 CN**: 继续一个多行参数列表、初始化器或聚合项：`OMP_BIND_parallel,`。
- **L92 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OMP_BIND_teams,`.
  **L92 CN**: 继续一个多行参数列表、初始化器或聚合项：`OMP_BIND_teams,`。
- **L93 EN**: Continues the surrounding expression or declaration: `OMP_BIND_thread`.
  **L93 CN**: 继续构造周围的表达式或声明：`OMP_BIND_thread`。
- **L94 EN**: Executes a standalone statement or declaration: `];`.
  **L94 CN**: 执行一条独立语句或声明：`];`。
- **L95 EN**: Closes the current lexical scope or compound statement.
  **L95 CN**: 结束当前词法作用域或复合语句块。
- **L96 EN**: Blank line separating nearby declarations or logic blocks.
  **L96 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 97-128

````tablegen
def OMP_CANCELLATION_CONSTRUCT_Parallel : EnumVal<"parallel", 1, 1> {}
def OMP_CANCELLATION_CONSTRUCT_Loop : EnumVal<"loop", 2, 1> {}
def OMP_CANCELLATION_CONSTRUCT_Sections : EnumVal<"sections", 3, 1> {}
def OMP_CANCELLATION_CONSTRUCT_Taskgroup : EnumVal<"taskgroup", 4, 1> {}
def OMP_CANCELLATION_CONSTRUCT_None : EnumVal<"none", 5, 0> {
  let isDefault = 1;
}
def OMPC_CancellationConstructType
    : Clause<[Spelling<"cancellation_construct_type">]> {
  let enumClauseValue = "CancellationConstructType";
  let allowedClauseValues = [
    OMP_CANCELLATION_CONSTRUCT_Parallel,
    OMP_CANCELLATION_CONSTRUCT_Loop,
    OMP_CANCELLATION_CONSTRUCT_Sections,
    OMP_CANCELLATION_CONSTRUCT_Taskgroup,
    OMP_CANCELLATION_CONSTRUCT_None
  ];
  let flangClass = "OmpCancellationConstructTypeClause";
  let skipFlangUnparser = true;
}
def OMPC_Capture : Clause<[Spelling<"capture">]> {
  let clangClass = "OMPCaptureClause";
}
def OMPC_Collapse : Clause<[Spelling<"collapse">]> {
  let clangClass = "OMPCollapseClause";
  let flangClass = "ScalarIntConstantExpr";
}
def OMPC_Collector : Clause<[Spelling<"collector">]> {
}
def OMPC_Combiner : Clause<[Spelling<"combiner">]> {
  let flangClass = "OmpCombinerClause";
}
````
- **L97 EN**: Declares TableGen def `OMP_CANCELLATION_CONSTRUCT_Parallel`.
  **L97 CN**: 声明 TableGen def `OMP_CANCELLATION_CONSTRUCT_Parallel`。
- **L98 EN**: Declares TableGen def `OMP_CANCELLATION_CONSTRUCT_Loop`.
  **L98 CN**: 声明 TableGen def `OMP_CANCELLATION_CONSTRUCT_Loop`。
- **L99 EN**: Declares TableGen def `OMP_CANCELLATION_CONSTRUCT_Sections`.
  **L99 CN**: 声明 TableGen def `OMP_CANCELLATION_CONSTRUCT_Sections`。
- **L100 EN**: Declares TableGen def `OMP_CANCELLATION_CONSTRUCT_Taskgroup`.
  **L100 CN**: 声明 TableGen def `OMP_CANCELLATION_CONSTRUCT_Taskgroup`。
- **L101 EN**: Declares TableGen def `OMP_CANCELLATION_CONSTRUCT_None`.
  **L101 CN**: 声明 TableGen def `OMP_CANCELLATION_CONSTRUCT_None`。
- **L102 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L102 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L103 EN**: Closes the current lexical scope or compound statement.
  **L103 CN**: 结束当前词法作用域或复合语句块。
- **L104 EN**: Declares TableGen def `OMPC_CancellationConstructType`.
  **L104 CN**: 声明 TableGen def `OMPC_CancellationConstructType`。
- **L105 EN**: Continues the surrounding expression or declaration: `: Clause<[Spelling<"cancellation_construct_type">]> {`.
  **L105 CN**: 继续构造周围的表达式或声明：`: Clause<[Spelling<"cancellation_construct_type">]> {`。
- **L106 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L106 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L107 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L107 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L108 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OMP_CANCELLATION_CONSTRUCT_Parallel,`.
  **L108 CN**: 继续一个多行参数列表、初始化器或聚合项：`OMP_CANCELLATION_CONSTRUCT_Parallel,`。
- **L109 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OMP_CANCELLATION_CONSTRUCT_Loop,`.
  **L109 CN**: 继续一个多行参数列表、初始化器或聚合项：`OMP_CANCELLATION_CONSTRUCT_Loop,`。
- **L110 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OMP_CANCELLATION_CONSTRUCT_Sections,`.
  **L110 CN**: 继续一个多行参数列表、初始化器或聚合项：`OMP_CANCELLATION_CONSTRUCT_Sections,`。
- **L111 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OMP_CANCELLATION_CONSTRUCT_Taskgroup,`.
  **L111 CN**: 继续一个多行参数列表、初始化器或聚合项：`OMP_CANCELLATION_CONSTRUCT_Taskgroup,`。
- **L112 EN**: Continues the surrounding expression or declaration: `OMP_CANCELLATION_CONSTRUCT_None`.
  **L112 CN**: 继续构造周围的表达式或声明：`OMP_CANCELLATION_CONSTRUCT_None`。
- **L113 EN**: Executes a standalone statement or declaration: `];`.
  **L113 CN**: 执行一条独立语句或声明：`];`。
- **L114 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L114 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L115 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L115 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L116 EN**: Closes the current lexical scope or compound statement.
  **L116 CN**: 结束当前词法作用域或复合语句块。
- **L117 EN**: Declares TableGen def `OMPC_Capture`.
  **L117 CN**: 声明 TableGen def `OMPC_Capture`。
- **L118 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L118 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L119 EN**: Closes the current lexical scope or compound statement.
  **L119 CN**: 结束当前词法作用域或复合语句块。
- **L120 EN**: Declares TableGen def `OMPC_Collapse`.
  **L120 CN**: 声明 TableGen def `OMPC_Collapse`。
- **L121 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L121 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L122 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L122 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L123 EN**: Closes the current lexical scope or compound statement.
  **L123 CN**: 结束当前词法作用域或复合语句块。
- **L124 EN**: Declares TableGen def `OMPC_Collector`.
  **L124 CN**: 声明 TableGen def `OMPC_Collector`。
- **L125 EN**: Closes the current lexical scope or compound statement.
  **L125 CN**: 结束当前词法作用域或复合语句块。
- **L126 EN**: Declares TableGen def `OMPC_Combiner`.
  **L126 CN**: 声明 TableGen def `OMPC_Combiner`。
- **L127 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L127 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L128 EN**: Closes the current lexical scope or compound statement.
  **L128 CN**: 结束当前词法作用域或复合语句块。

### Lines 129-160

````tablegen
def OMPC_Compare : Clause<[Spelling<"compare">]> {
  let clangClass = "OMPCompareClause";
}
def OMPC_Contains : Clause<[Spelling<"contains">]> {
  let clangClass = "OMPContainsClause";
  let flangClass = "OmpContainsClause";
}
def OMPC_Copyin : Clause<[Spelling<"copyin">]> {
  let clangClass = "OMPCopyinClause";
  let flangClass = "OmpObjectList";
}
def OMPC_CopyPrivate : Clause<[Spelling<"copyprivate">]> {
  let clangClass = "OMPCopyprivateClause";
  let flangClass = "OmpObjectList";
}
def OMPC_Counts : Clause<[Spelling<"counts">]> {
  let clangClass = "OMPCountsClause";
}
def OMPC_Default : Clause<[Spelling<"default">]> {
  let clangClass = "OMPDefaultClause";
  let flangClass = "OmpDefaultClause";
}
def OMPC_DefaultMap : Clause<[Spelling<"defaultmap">]> {
  let clangClass = "OMPDefaultmapClause";
  let flangClass = "OmpDefaultmapClause";
}
def OMPC_Depend : Clause<[Spelling<"depend">]> {
  let clangClass = "OMPDependClause";
  let flangClass = "OmpDependClause";
}
def OMPC_Depobj : Clause<[Spelling<"depobj">]> {
  let clangClass = "OMPDepobjClause";
````
- **L129 EN**: Declares TableGen def `OMPC_Compare`.
  **L129 CN**: 声明 TableGen def `OMPC_Compare`。
- **L130 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L130 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L131 EN**: Closes the current lexical scope or compound statement.
  **L131 CN**: 结束当前词法作用域或复合语句块。
- **L132 EN**: Declares TableGen def `OMPC_Contains`.
  **L132 CN**: 声明 TableGen def `OMPC_Contains`。
- **L133 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L133 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L134 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L134 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L135 EN**: Closes the current lexical scope or compound statement.
  **L135 CN**: 结束当前词法作用域或复合语句块。
- **L136 EN**: Declares TableGen def `OMPC_Copyin`.
  **L136 CN**: 声明 TableGen def `OMPC_Copyin`。
- **L137 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L137 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L138 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L138 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L139 EN**: Closes the current lexical scope or compound statement.
  **L139 CN**: 结束当前词法作用域或复合语句块。
- **L140 EN**: Declares TableGen def `OMPC_CopyPrivate`.
  **L140 CN**: 声明 TableGen def `OMPC_CopyPrivate`。
- **L141 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L141 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L142 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L142 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L143 EN**: Closes the current lexical scope or compound statement.
  **L143 CN**: 结束当前词法作用域或复合语句块。
- **L144 EN**: Declares TableGen def `OMPC_Counts`.
  **L144 CN**: 声明 TableGen def `OMPC_Counts`。
- **L145 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L145 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L146 EN**: Closes the current lexical scope or compound statement.
  **L146 CN**: 结束当前词法作用域或复合语句块。
- **L147 EN**: Declares TableGen def `OMPC_Default`.
  **L147 CN**: 声明 TableGen def `OMPC_Default`。
- **L148 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L148 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L149 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L149 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L150 EN**: Closes the current lexical scope or compound statement.
  **L150 CN**: 结束当前词法作用域或复合语句块。
- **L151 EN**: Declares TableGen def `OMPC_DefaultMap`.
  **L151 CN**: 声明 TableGen def `OMPC_DefaultMap`。
- **L152 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L152 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L153 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L153 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L154 EN**: Closes the current lexical scope or compound statement.
  **L154 CN**: 结束当前词法作用域或复合语句块。
- **L155 EN**: Declares TableGen def `OMPC_Depend`.
  **L155 CN**: 声明 TableGen def `OMPC_Depend`。
- **L156 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L156 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L157 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L157 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L158 EN**: Closes the current lexical scope or compound statement.
  **L158 CN**: 结束当前词法作用域或复合语句块。
- **L159 EN**: Declares TableGen def `OMPC_Depobj`.
  **L159 CN**: 声明 TableGen def `OMPC_Depobj`。
- **L160 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L160 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 161-192

````tablegen
  let isImplicit = true;
}
def OMPC_Depth : Clause<[Spelling<"depth">]> {
  let flangClass = "ScalarIntConstantExpr";
}
def OMPC_Destroy : Clause<[Spelling<"destroy">]> {
  let clangClass = "OMPDestroyClause";
  let flangClass = "OmpDestroyClause";
  let isValueOptional = true;
}
def OMPC_Detach : Clause<[Spelling<"detach">]> {
  let clangClass = "OMPDetachClause";
  let flangClass = "OmpDetachClause";
}
def OMPC_Device : Clause<[Spelling<"device">]> {
  let clangClass = "OMPDeviceClause";
  let flangClass = "OmpDeviceClause";
}
def OMPC_DeviceSafesync : Clause<[Spelling<"device_safesync">]> {
  let flangClass = "OmpDeviceSafesyncClause";
  let isValueOptional = true;
}
def OMPC_DeviceType : Clause<[Spelling<"device_type">]> {
  let flangClass = "OmpDeviceTypeClause";
}
def OMPC_DistSchedule : Clause<[Spelling<"dist_schedule">]> {
  let clangClass = "OMPDistScheduleClause";
  let flangClass = "ScalarIntExpr";
  let isValueOptional = true;
}
def OMPC_Doacross : Clause<[Spelling<"doacross">]> {
  let clangClass = "OMPDoacrossClause";
````
- **L161 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L161 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L162 EN**: Closes the current lexical scope or compound statement.
  **L162 CN**: 结束当前词法作用域或复合语句块。
- **L163 EN**: Declares TableGen def `OMPC_Depth`.
  **L163 CN**: 声明 TableGen def `OMPC_Depth`。
- **L164 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L164 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L165 EN**: Closes the current lexical scope or compound statement.
  **L165 CN**: 结束当前词法作用域或复合语句块。
- **L166 EN**: Declares TableGen def `OMPC_Destroy`.
  **L166 CN**: 声明 TableGen def `OMPC_Destroy`。
- **L167 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L167 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L168 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L168 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L169 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L169 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L170 EN**: Closes the current lexical scope or compound statement.
  **L170 CN**: 结束当前词法作用域或复合语句块。
- **L171 EN**: Declares TableGen def `OMPC_Detach`.
  **L171 CN**: 声明 TableGen def `OMPC_Detach`。
- **L172 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L172 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L173 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L173 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L174 EN**: Closes the current lexical scope or compound statement.
  **L174 CN**: 结束当前词法作用域或复合语句块。
- **L175 EN**: Declares TableGen def `OMPC_Device`.
  **L175 CN**: 声明 TableGen def `OMPC_Device`。
- **L176 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L176 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L177 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L177 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L178 EN**: Closes the current lexical scope or compound statement.
  **L178 CN**: 结束当前词法作用域或复合语句块。
- **L179 EN**: Declares TableGen def `OMPC_DeviceSafesync`.
  **L179 CN**: 声明 TableGen def `OMPC_DeviceSafesync`。
- **L180 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L180 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L181 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L181 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L182 EN**: Closes the current lexical scope or compound statement.
  **L182 CN**: 结束当前词法作用域或复合语句块。
- **L183 EN**: Declares TableGen def `OMPC_DeviceType`.
  **L183 CN**: 声明 TableGen def `OMPC_DeviceType`。
- **L184 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L184 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L185 EN**: Closes the current lexical scope or compound statement.
  **L185 CN**: 结束当前词法作用域或复合语句块。
- **L186 EN**: Declares TableGen def `OMPC_DistSchedule`.
  **L186 CN**: 声明 TableGen def `OMPC_DistSchedule`。
- **L187 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L187 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L188 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L188 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L189 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L189 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L190 EN**: Closes the current lexical scope or compound statement.
  **L190 CN**: 结束当前词法作用域或复合语句块。
- **L191 EN**: Declares TableGen def `OMPC_Doacross`.
  **L191 CN**: 声明 TableGen def `OMPC_Doacross`。
- **L192 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L192 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 193-224

````tablegen
  let flangClass = "OmpDoacrossClause";
}
def OMPC_DynamicAllocators : Clause<[Spelling<"dynamic_allocators">]> {
  let clangClass = "OMPDynamicAllocatorsClause";
  let flangClass = "OmpDynamicAllocatorsClause";
  let isValueOptional = true;
}
def OMPC_DynGroupprivate : Clause<[Spelling<"dyn_groupprivate">]> {
  let clangClass = "OMPDynGroupprivateClause";
  let flangClass = "OmpDynGroupprivateClause";
}
def OMPC_Enter : Clause<[Spelling<"enter">]> {
  let flangClass = "OmpEnterClause";
}
def OMPC_Exclusive : Clause<[Spelling<"exclusive">]> {
  let clangClass = "OMPExclusiveClause";
  let flangClass = "OmpObjectList";
}
def OMPC_Fail : Clause<[Spelling<"fail">]> {
  let clangClass = "OMPFailClause";
  let flangClass = "OmpFailClause";
}
def OMPC_Filter : Clause<[Spelling<"filter">]> {
  let clangClass = "OMPFilterClause";
  let flangClass = "ScalarIntExpr";
}
def OMPC_Final : Clause<[Spelling<"final">]> {
  let clangClass = "OMPFinalClause";
  let flangClass = "ScalarLogicalExpr";
}
def OMPC_FirstPrivate : Clause<[Spelling<"firstprivate">]> {
  let clangClass = "OMPFirstprivateClause";
````
- **L193 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L193 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L194 EN**: Closes the current lexical scope or compound statement.
  **L194 CN**: 结束当前词法作用域或复合语句块。
- **L195 EN**: Declares TableGen def `OMPC_DynamicAllocators`.
  **L195 CN**: 声明 TableGen def `OMPC_DynamicAllocators`。
- **L196 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L196 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L197 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L197 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L198 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L198 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L199 EN**: Closes the current lexical scope or compound statement.
  **L199 CN**: 结束当前词法作用域或复合语句块。
- **L200 EN**: Declares TableGen def `OMPC_DynGroupprivate`.
  **L200 CN**: 声明 TableGen def `OMPC_DynGroupprivate`。
- **L201 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L201 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L202 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L202 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L203 EN**: Closes the current lexical scope or compound statement.
  **L203 CN**: 结束当前词法作用域或复合语句块。
- **L204 EN**: Declares TableGen def `OMPC_Enter`.
  **L204 CN**: 声明 TableGen def `OMPC_Enter`。
- **L205 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L205 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L206 EN**: Closes the current lexical scope or compound statement.
  **L206 CN**: 结束当前词法作用域或复合语句块。
- **L207 EN**: Declares TableGen def `OMPC_Exclusive`.
  **L207 CN**: 声明 TableGen def `OMPC_Exclusive`。
- **L208 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L208 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L209 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L209 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L210 EN**: Closes the current lexical scope or compound statement.
  **L210 CN**: 结束当前词法作用域或复合语句块。
- **L211 EN**: Declares TableGen def `OMPC_Fail`.
  **L211 CN**: 声明 TableGen def `OMPC_Fail`。
- **L212 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L212 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L213 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L213 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L214 EN**: Closes the current lexical scope or compound statement.
  **L214 CN**: 结束当前词法作用域或复合语句块。
- **L215 EN**: Declares TableGen def `OMPC_Filter`.
  **L215 CN**: 声明 TableGen def `OMPC_Filter`。
- **L216 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L216 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L217 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L217 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L218 EN**: Closes the current lexical scope or compound statement.
  **L218 CN**: 结束当前词法作用域或复合语句块。
- **L219 EN**: Declares TableGen def `OMPC_Final`.
  **L219 CN**: 声明 TableGen def `OMPC_Final`。
- **L220 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L220 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L221 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L221 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L222 EN**: Closes the current lexical scope or compound statement.
  **L222 CN**: 结束当前词法作用域或复合语句块。
- **L223 EN**: Declares TableGen def `OMPC_FirstPrivate`.
  **L223 CN**: 声明 TableGen def `OMPC_FirstPrivate`。
- **L224 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L224 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 225-256

````tablegen
  let flangClass = "OmpObjectList";
}
def OMPC_Flush : Clause<[Spelling<"flush">]> {
  let clangClass = "OMPFlushClause";
  let isImplicit = true;
}
def OMPC_From : Clause<[Spelling<"from">]> {
  let clangClass = "OMPFromClause";
  let flangClass = "OmpFromClause";
}
def OMPC_Full: Clause<[Spelling<"full">]> {
  let clangClass = "OMPFullClause";
}
def OMP_GRAINSIZE_Strict : EnumVal<"strict", 1, 1> {}
def OMP_GRAINSIZE_Unknown : EnumVal<"unknown", 2, 0> { let isDefault = 1; }
def OMPC_GrainSize : Clause<[Spelling<"grainsize">]> {
  let clangClass = "OMPGrainsizeClause";
  let flangClass = "OmpGrainsizeClause";
  let enumClauseValue = "GrainsizeType";
  let allowedClauseValues = [
    OMP_GRAINSIZE_Strict,
    OMP_GRAINSIZE_Unknown
  ];
}
def OMPC_GraphId : Clause<[Spelling<"graph_id">]> {
  let flangClass = "OmpGraphIdClause";
}
def OMPC_GraphReset : Clause<[Spelling<"graph_reset">]> {
  let flangClass = "OmpGraphResetClause";
  let isValueOptional = true;
}
def OMPC_GroupPrivate : Clause<[Spelling<"groupprivate">]> {
````
- **L225 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L225 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L226 EN**: Closes the current lexical scope or compound statement.
  **L226 CN**: 结束当前词法作用域或复合语句块。
- **L227 EN**: Declares TableGen def `OMPC_Flush`.
  **L227 CN**: 声明 TableGen def `OMPC_Flush`。
- **L228 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L228 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L229 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L229 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L230 EN**: Closes the current lexical scope or compound statement.
  **L230 CN**: 结束当前词法作用域或复合语句块。
- **L231 EN**: Declares TableGen def `OMPC_From`.
  **L231 CN**: 声明 TableGen def `OMPC_From`。
- **L232 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L232 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L233 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L233 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L234 EN**: Closes the current lexical scope or compound statement.
  **L234 CN**: 结束当前词法作用域或复合语句块。
- **L235 EN**: Declares TableGen def `OMPC_Full`.
  **L235 CN**: 声明 TableGen def `OMPC_Full`。
- **L236 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L236 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L237 EN**: Closes the current lexical scope or compound statement.
  **L237 CN**: 结束当前词法作用域或复合语句块。
- **L238 EN**: Declares TableGen def `OMP_GRAINSIZE_Strict`.
  **L238 CN**: 声明 TableGen def `OMP_GRAINSIZE_Strict`。
- **L239 EN**: Declares TableGen def `OMP_GRAINSIZE_Unknown`.
  **L239 CN**: 声明 TableGen def `OMP_GRAINSIZE_Unknown`。
- **L240 EN**: Declares TableGen def `OMPC_GrainSize`.
  **L240 CN**: 声明 TableGen def `OMPC_GrainSize`。
- **L241 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L241 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L242 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L242 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L243 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L243 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L244 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L244 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L245 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OMP_GRAINSIZE_Strict,`.
  **L245 CN**: 继续一个多行参数列表、初始化器或聚合项：`OMP_GRAINSIZE_Strict,`。
- **L246 EN**: Continues the surrounding expression or declaration: `OMP_GRAINSIZE_Unknown`.
  **L246 CN**: 继续构造周围的表达式或声明：`OMP_GRAINSIZE_Unknown`。
- **L247 EN**: Executes a standalone statement or declaration: `];`.
  **L247 CN**: 执行一条独立语句或声明：`];`。
- **L248 EN**: Closes the current lexical scope or compound statement.
  **L248 CN**: 结束当前词法作用域或复合语句块。
- **L249 EN**: Declares TableGen def `OMPC_GraphId`.
  **L249 CN**: 声明 TableGen def `OMPC_GraphId`。
- **L250 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L250 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L251 EN**: Closes the current lexical scope or compound statement.
  **L251 CN**: 结束当前词法作用域或复合语句块。
- **L252 EN**: Declares TableGen def `OMPC_GraphReset`.
  **L252 CN**: 声明 TableGen def `OMPC_GraphReset`。
- **L253 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L253 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L254 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L254 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L255 EN**: Closes the current lexical scope or compound statement.
  **L255 CN**: 结束当前词法作用域或复合语句块。
- **L256 EN**: Declares TableGen def `OMPC_GroupPrivate`.
  **L256 CN**: 声明 TableGen def `OMPC_GroupPrivate`。

### Lines 257-288

````tablegen
  let isImplicit = true;
}
def OMPC_HasDeviceAddr : Clause<[Spelling<"has_device_addr">]> {
  let clangClass = "OMPHasDeviceAddrClause";
  let flangClass = "OmpObjectList";
}
def OMPC_Hint : Clause<[Spelling<"hint">]> {
  let clangClass = "OMPHintClause";
  let flangClass = "OmpHintClause";
}
def OMPC_Holds : Clause<[Spelling<"holds">]> {
  let clangClass = "OMPHoldsClause";
  let flangClass = "OmpHoldsClause";
}
def OMPC_If : Clause<[Spelling<"if">]> {
  let clangClass = "OMPIfClause";
  let flangClass = "OmpIfClause";
}
def OMPC_Inbranch : Clause<[Spelling<"inbranch">]> {
}
def OMPC_Inclusive : Clause<[Spelling<"inclusive">]> {
  let clangClass = "OMPInclusiveClause";
  let flangClass = "OmpObjectList";
}
def OMPC_Indirect : Clause<[Spelling<"indirect">]> {
  let flangClass = "OmpIndirectClause";
}
def OMPC_Induction : Clause<[Spelling<"induction">]> {
}
def OMPC_Inductor : Clause<[Spelling<"inductor">]> {
}
def OMPC_Init : Clause<[Spelling<"init">]> {
````
- **L257 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L257 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L258 EN**: Closes the current lexical scope or compound statement.
  **L258 CN**: 结束当前词法作用域或复合语句块。
- **L259 EN**: Declares TableGen def `OMPC_HasDeviceAddr`.
  **L259 CN**: 声明 TableGen def `OMPC_HasDeviceAddr`。
- **L260 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L260 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L261 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L261 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L262 EN**: Closes the current lexical scope or compound statement.
  **L262 CN**: 结束当前词法作用域或复合语句块。
- **L263 EN**: Declares TableGen def `OMPC_Hint`.
  **L263 CN**: 声明 TableGen def `OMPC_Hint`。
- **L264 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L264 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L265 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L265 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L266 EN**: Closes the current lexical scope or compound statement.
  **L266 CN**: 结束当前词法作用域或复合语句块。
- **L267 EN**: Declares TableGen def `OMPC_Holds`.
  **L267 CN**: 声明 TableGen def `OMPC_Holds`。
- **L268 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L268 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L269 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L269 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L270 EN**: Closes the current lexical scope or compound statement.
  **L270 CN**: 结束当前词法作用域或复合语句块。
- **L271 EN**: Declares TableGen def `OMPC_If`.
  **L271 CN**: 声明 TableGen def `OMPC_If`。
- **L272 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L272 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L273 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L273 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L274 EN**: Closes the current lexical scope or compound statement.
  **L274 CN**: 结束当前词法作用域或复合语句块。
- **L275 EN**: Declares TableGen def `OMPC_Inbranch`.
  **L275 CN**: 声明 TableGen def `OMPC_Inbranch`。
- **L276 EN**: Closes the current lexical scope or compound statement.
  **L276 CN**: 结束当前词法作用域或复合语句块。
- **L277 EN**: Declares TableGen def `OMPC_Inclusive`.
  **L277 CN**: 声明 TableGen def `OMPC_Inclusive`。
- **L278 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L278 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L279 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L279 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L280 EN**: Closes the current lexical scope or compound statement.
  **L280 CN**: 结束当前词法作用域或复合语句块。
- **L281 EN**: Declares TableGen def `OMPC_Indirect`.
  **L281 CN**: 声明 TableGen def `OMPC_Indirect`。
- **L282 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L282 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L283 EN**: Closes the current lexical scope or compound statement.
  **L283 CN**: 结束当前词法作用域或复合语句块。
- **L284 EN**: Declares TableGen def `OMPC_Induction`.
  **L284 CN**: 声明 TableGen def `OMPC_Induction`。
- **L285 EN**: Closes the current lexical scope or compound statement.
  **L285 CN**: 结束当前词法作用域或复合语句块。
- **L286 EN**: Declares TableGen def `OMPC_Inductor`.
  **L286 CN**: 声明 TableGen def `OMPC_Inductor`。
- **L287 EN**: Closes the current lexical scope or compound statement.
  **L287 CN**: 结束当前词法作用域或复合语句块。
- **L288 EN**: Declares TableGen def `OMPC_Init`.
  **L288 CN**: 声明 TableGen def `OMPC_Init`。

### Lines 289-320

````tablegen
  let clangClass = "OMPInitClause";
  let flangClass = "OmpInitClause";
}
def OMPC_InitComplete : Clause<[Spelling<"init_complete">]> {
  let isValueOptional = true;
}
def OMPC_Initializer : Clause<[Spelling<"initializer">]> {
  let flangClass = "OmpInitializerClause";
}
def OMPC_InReduction : Clause<[Spelling<"in_reduction">]> {
  let clangClass = "OMPInReductionClause";
  let flangClass = "OmpInReductionClause";
}
def OMPC_Interop : Clause<[Spelling<"interop">]> {
}
def OMPC_IsDevicePtr : Clause<[Spelling<"is_device_ptr">]> {
  let clangClass = "OMPIsDevicePtrClause";
  let flangClass = "OmpObjectList";
}
def OMPC_LastPrivate : Clause<[Spelling<"lastprivate">]> {
  let clangClass = "OMPLastprivateClause";
  let flangClass = "OmpLastprivateClause";
}
def OMPC_Linear : Clause<[Spelling<"linear">]> {
  let clangClass = "OMPLinearClause";
  let flangClass = "OmpLinearClause";
}
def OMPC_Link : Clause<[Spelling<"link">]> {
  let flangClass = "OmpObjectList";
}
def OMPC_Local : Clause<[Spelling<"local">]> {
}
````
- **L289 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L289 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L290 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L290 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L291 EN**: Closes the current lexical scope or compound statement.
  **L291 CN**: 结束当前词法作用域或复合语句块。
- **L292 EN**: Declares TableGen def `OMPC_InitComplete`.
  **L292 CN**: 声明 TableGen def `OMPC_InitComplete`。
- **L293 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L293 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L294 EN**: Closes the current lexical scope or compound statement.
  **L294 CN**: 结束当前词法作用域或复合语句块。
- **L295 EN**: Declares TableGen def `OMPC_Initializer`.
  **L295 CN**: 声明 TableGen def `OMPC_Initializer`。
- **L296 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L296 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L297 EN**: Closes the current lexical scope or compound statement.
  **L297 CN**: 结束当前词法作用域或复合语句块。
- **L298 EN**: Declares TableGen def `OMPC_InReduction`.
  **L298 CN**: 声明 TableGen def `OMPC_InReduction`。
- **L299 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L299 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L300 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L300 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L301 EN**: Closes the current lexical scope or compound statement.
  **L301 CN**: 结束当前词法作用域或复合语句块。
- **L302 EN**: Declares TableGen def `OMPC_Interop`.
  **L302 CN**: 声明 TableGen def `OMPC_Interop`。
- **L303 EN**: Closes the current lexical scope or compound statement.
  **L303 CN**: 结束当前词法作用域或复合语句块。
- **L304 EN**: Declares TableGen def `OMPC_IsDevicePtr`.
  **L304 CN**: 声明 TableGen def `OMPC_IsDevicePtr`。
- **L305 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L305 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L306 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L306 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L307 EN**: Closes the current lexical scope or compound statement.
  **L307 CN**: 结束当前词法作用域或复合语句块。
- **L308 EN**: Declares TableGen def `OMPC_LastPrivate`.
  **L308 CN**: 声明 TableGen def `OMPC_LastPrivate`。
- **L309 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L309 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L310 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L310 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L311 EN**: Closes the current lexical scope or compound statement.
  **L311 CN**: 结束当前词法作用域或复合语句块。
- **L312 EN**: Declares TableGen def `OMPC_Linear`.
  **L312 CN**: 声明 TableGen def `OMPC_Linear`。
- **L313 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L313 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L314 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L314 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L315 EN**: Closes the current lexical scope or compound statement.
  **L315 CN**: 结束当前词法作用域或复合语句块。
- **L316 EN**: Declares TableGen def `OMPC_Link`.
  **L316 CN**: 声明 TableGen def `OMPC_Link`。
- **L317 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L317 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L318 EN**: Closes the current lexical scope or compound statement.
  **L318 CN**: 结束当前词法作用域或复合语句块。
- **L319 EN**: Declares TableGen def `OMPC_Local`.
  **L319 CN**: 声明 TableGen def `OMPC_Local`。
- **L320 EN**: Closes the current lexical scope or compound statement.
  **L320 CN**: 结束当前词法作用域或复合语句块。

### Lines 321-352

````tablegen
def OMPC_LoopRange : Clause<[Spelling<"looprange">]> {
  let clangClass = "OMPLoopRangeClause";
  let flangClass = "OmpLooprangeClause";
}
def OMPC_Map : Clause<[Spelling<"map">]> {
  let clangClass = "OMPMapClause";
  let flangClass = "OmpMapClause";
}
def OMPC_Match : Clause<[Spelling<"match">]> {
  let flangClass = "OmpMatchClause";
}
def OMPC_Memscope : Clause<[Spelling<"memscope">]> {
}
def OMP_MEMORY_ORDER_SeqCst : EnumVal<"seq_cst", 1, 1> {}
def OMP_MEMORY_ORDER_AcqRel : EnumVal<"acq_rel", 2, 1> {}
def OMP_MEMORY_ORDER_Acquire : EnumVal<"acquire", 3, 1> {}
def OMP_MEMORY_ORDER_Release : EnumVal<"release", 4, 1> {}
def OMP_MEMORY_ORDER_Relaxed : EnumVal<"relaxed", 5, 1> {}
def OMP_MEMORY_ORDER_Default : EnumVal<"default", 6, 0> {
  let isDefault = 1;
}
def OMPC_MemoryOrder : Clause<[Spelling<"memory_order">]> {
  let enumClauseValue = "MemoryOrderKind";
  let allowedClauseValues = [
    OMP_MEMORY_ORDER_SeqCst,
    OMP_MEMORY_ORDER_AcqRel,
    OMP_MEMORY_ORDER_Acquire,
    OMP_MEMORY_ORDER_Release,
    OMP_MEMORY_ORDER_Relaxed,
    OMP_MEMORY_ORDER_Default
  ];
}
````
- **L321 EN**: Declares TableGen def `OMPC_LoopRange`.
  **L321 CN**: 声明 TableGen def `OMPC_LoopRange`。
- **L322 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L322 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L323 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L323 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L324 EN**: Closes the current lexical scope or compound statement.
  **L324 CN**: 结束当前词法作用域或复合语句块。
- **L325 EN**: Declares TableGen def `OMPC_Map`.
  **L325 CN**: 声明 TableGen def `OMPC_Map`。
- **L326 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L326 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L327 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L327 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L328 EN**: Closes the current lexical scope or compound statement.
  **L328 CN**: 结束当前词法作用域或复合语句块。
- **L329 EN**: Declares TableGen def `OMPC_Match`.
  **L329 CN**: 声明 TableGen def `OMPC_Match`。
- **L330 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L330 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L331 EN**: Closes the current lexical scope or compound statement.
  **L331 CN**: 结束当前词法作用域或复合语句块。
- **L332 EN**: Declares TableGen def `OMPC_Memscope`.
  **L332 CN**: 声明 TableGen def `OMPC_Memscope`。
- **L333 EN**: Closes the current lexical scope or compound statement.
  **L333 CN**: 结束当前词法作用域或复合语句块。
- **L334 EN**: Declares TableGen def `OMP_MEMORY_ORDER_SeqCst`.
  **L334 CN**: 声明 TableGen def `OMP_MEMORY_ORDER_SeqCst`。
- **L335 EN**: Declares TableGen def `OMP_MEMORY_ORDER_AcqRel`.
  **L335 CN**: 声明 TableGen def `OMP_MEMORY_ORDER_AcqRel`。
- **L336 EN**: Declares TableGen def `OMP_MEMORY_ORDER_Acquire`.
  **L336 CN**: 声明 TableGen def `OMP_MEMORY_ORDER_Acquire`。
- **L337 EN**: Declares TableGen def `OMP_MEMORY_ORDER_Release`.
  **L337 CN**: 声明 TableGen def `OMP_MEMORY_ORDER_Release`。
- **L338 EN**: Declares TableGen def `OMP_MEMORY_ORDER_Relaxed`.
  **L338 CN**: 声明 TableGen def `OMP_MEMORY_ORDER_Relaxed`。
- **L339 EN**: Declares TableGen def `OMP_MEMORY_ORDER_Default`.
  **L339 CN**: 声明 TableGen def `OMP_MEMORY_ORDER_Default`。
- **L340 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L340 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L341 EN**: Closes the current lexical scope or compound statement.
  **L341 CN**: 结束当前词法作用域或复合语句块。
- **L342 EN**: Declares TableGen def `OMPC_MemoryOrder`.
  **L342 CN**: 声明 TableGen def `OMPC_MemoryOrder`。
- **L343 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L343 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L344 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L344 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L345 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OMP_MEMORY_ORDER_SeqCst,`.
  **L345 CN**: 继续一个多行参数列表、初始化器或聚合项：`OMP_MEMORY_ORDER_SeqCst,`。
- **L346 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OMP_MEMORY_ORDER_AcqRel,`.
  **L346 CN**: 继续一个多行参数列表、初始化器或聚合项：`OMP_MEMORY_ORDER_AcqRel,`。
- **L347 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OMP_MEMORY_ORDER_Acquire,`.
  **L347 CN**: 继续一个多行参数列表、初始化器或聚合项：`OMP_MEMORY_ORDER_Acquire,`。
- **L348 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OMP_MEMORY_ORDER_Release,`.
  **L348 CN**: 继续一个多行参数列表、初始化器或聚合项：`OMP_MEMORY_ORDER_Release,`。
- **L349 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OMP_MEMORY_ORDER_Relaxed,`.
  **L349 CN**: 继续一个多行参数列表、初始化器或聚合项：`OMP_MEMORY_ORDER_Relaxed,`。
- **L350 EN**: Continues the surrounding expression or declaration: `OMP_MEMORY_ORDER_Default`.
  **L350 CN**: 继续构造周围的表达式或声明：`OMP_MEMORY_ORDER_Default`。
- **L351 EN**: Executes a standalone statement or declaration: `];`.
  **L351 CN**: 执行一条独立语句或声明：`];`。
- **L352 EN**: Closes the current lexical scope or compound statement.
  **L352 CN**: 结束当前词法作用域或复合语句块。

### Lines 353-384

````tablegen
def OMPC_Mergeable : Clause<[Spelling<"mergeable">]> {
  let clangClass = "OMPMergeableClause";
}
def OMPC_Message : Clause<[Spelling<"message">]> {
  let clangClass = "OMPMessageClause";
  let flangClass = "OmpMessageClause";
}
def OMPC_NoOpenMP : Clause<[Spelling<"no_openmp">]> {
  let clangClass = "OMPNoOpenMPClause";
}
def OMPC_NoOpenMPConstructs : Clause<[Spelling<"no_openmp_constructs">]> {
  let clangClass = "OMPNoOpenMPConstructsClause";
}
def OMPC_NoOpenMPRoutines : Clause<[Spelling<"no_openmp_routines">]> {
  let clangClass = "OMPNoOpenMPRoutinesClause";
}
def OMPC_NoParallelism : Clause<[Spelling<"no_parallelism">]> {
  let clangClass = "OMPNoParallelismClause";
}
def OMPC_Nocontext : Clause<[Spelling<"nocontext">]> {
  let clangClass = "OMPNocontextClause";
  let flangClass = "ScalarLogicalExpr";
}
def OMPC_NoGroup : Clause<[Spelling<"nogroup">]> {
  let clangClass = "OMPNogroupClause";
}
def OMPC_NonTemporal : Clause<[Spelling<"nontemporal">]> {
  let clangClass = "OMPNontemporalClause";
  let flangClass = "OmpObjectList";
}
def OMPC_Notinbranch : Clause<[Spelling<"notinbranch">]> {
}
````
- **L353 EN**: Declares TableGen def `OMPC_Mergeable`.
  **L353 CN**: 声明 TableGen def `OMPC_Mergeable`。
- **L354 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L354 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L355 EN**: Closes the current lexical scope or compound statement.
  **L355 CN**: 结束当前词法作用域或复合语句块。
- **L356 EN**: Declares TableGen def `OMPC_Message`.
  **L356 CN**: 声明 TableGen def `OMPC_Message`。
- **L357 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L357 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L358 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L358 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L359 EN**: Closes the current lexical scope or compound statement.
  **L359 CN**: 结束当前词法作用域或复合语句块。
- **L360 EN**: Declares TableGen def `OMPC_NoOpenMP`.
  **L360 CN**: 声明 TableGen def `OMPC_NoOpenMP`。
- **L361 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L361 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L362 EN**: Closes the current lexical scope or compound statement.
  **L362 CN**: 结束当前词法作用域或复合语句块。
- **L363 EN**: Declares TableGen def `OMPC_NoOpenMPConstructs`.
  **L363 CN**: 声明 TableGen def `OMPC_NoOpenMPConstructs`。
- **L364 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L364 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L365 EN**: Closes the current lexical scope or compound statement.
  **L365 CN**: 结束当前词法作用域或复合语句块。
- **L366 EN**: Declares TableGen def `OMPC_NoOpenMPRoutines`.
  **L366 CN**: 声明 TableGen def `OMPC_NoOpenMPRoutines`。
- **L367 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L367 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L368 EN**: Closes the current lexical scope or compound statement.
  **L368 CN**: 结束当前词法作用域或复合语句块。
- **L369 EN**: Declares TableGen def `OMPC_NoParallelism`.
  **L369 CN**: 声明 TableGen def `OMPC_NoParallelism`。
- **L370 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L370 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L371 EN**: Closes the current lexical scope or compound statement.
  **L371 CN**: 结束当前词法作用域或复合语句块。
- **L372 EN**: Declares TableGen def `OMPC_Nocontext`.
  **L372 CN**: 声明 TableGen def `OMPC_Nocontext`。
- **L373 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L373 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L374 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L374 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L375 EN**: Closes the current lexical scope or compound statement.
  **L375 CN**: 结束当前词法作用域或复合语句块。
- **L376 EN**: Declares TableGen def `OMPC_NoGroup`.
  **L376 CN**: 声明 TableGen def `OMPC_NoGroup`。
- **L377 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L377 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L378 EN**: Closes the current lexical scope or compound statement.
  **L378 CN**: 结束当前词法作用域或复合语句块。
- **L379 EN**: Declares TableGen def `OMPC_NonTemporal`.
  **L379 CN**: 声明 TableGen def `OMPC_NonTemporal`。
- **L380 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L380 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L381 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L381 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L382 EN**: Closes the current lexical scope or compound statement.
  **L382 CN**: 结束当前词法作用域或复合语句块。
- **L383 EN**: Declares TableGen def `OMPC_Notinbranch`.
  **L383 CN**: 声明 TableGen def `OMPC_Notinbranch`。
- **L384 EN**: Closes the current lexical scope or compound statement.
  **L384 CN**: 结束当前词法作用域或复合语句块。

### Lines 385-416

````tablegen
def OMPC_Novariants : Clause<[Spelling<"novariants">]> {
  let clangClass = "OMPNovariantsClause";
  let flangClass = "ScalarLogicalExpr";
}
def OMPC_NoWait : Clause<[Spelling<"nowait">]> {
  let clangClass = "OMPNowaitClause";
  let isValueOptional = true;
}
def OMP_NUMTASKS_Strict : EnumVal<"strict", 1, 1> {}
def OMP_NUMTASKS_Unknown : EnumVal<"unknown", 2, 0> { let isDefault = 1; }
def OMPC_NumTasks : Clause<[Spelling<"num_tasks">]> {
  let clangClass = "OMPNumTasksClause";
  let flangClass = "OmpNumTasksClause";
  let enumClauseValue = "NumTasksType";
  let allowedClauseValues = [
    OMP_NUMTASKS_Strict,
    OMP_NUMTASKS_Unknown
  ];
}
def OMPC_NumTeams : Clause<[Spelling<"num_teams">]> {
  let clangClass = "OMPNumTeamsClause";
  let flangClass = "OmpNumTeamsClause";
}
def OMP_NUMTHREADS_Strict : EnumVal<"strict", 1, 1> {}
def OMP_NUMTHREADS_Unknown : EnumVal<"unknown", 2, 0> { let isDefault = 1; }
def OMPC_NumThreads : Clause<[Spelling<"num_threads">]> {
  let clangClass = "OMPNumThreadsClause";
  let flangClass = "OmpNumThreadsClause";
  let enumClauseValue = "NumThreadsType";
  let allowedClauseValues = [
    OMP_NUMTHREADS_Strict,
    OMP_NUMTHREADS_Unknown
````
- **L385 EN**: Declares TableGen def `OMPC_Novariants`.
  **L385 CN**: 声明 TableGen def `OMPC_Novariants`。
- **L386 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L386 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L387 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L387 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L388 EN**: Closes the current lexical scope or compound statement.
  **L388 CN**: 结束当前词法作用域或复合语句块。
- **L389 EN**: Declares TableGen def `OMPC_NoWait`.
  **L389 CN**: 声明 TableGen def `OMPC_NoWait`。
- **L390 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L390 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L391 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L391 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L392 EN**: Closes the current lexical scope or compound statement.
  **L392 CN**: 结束当前词法作用域或复合语句块。
- **L393 EN**: Declares TableGen def `OMP_NUMTASKS_Strict`.
  **L393 CN**: 声明 TableGen def `OMP_NUMTASKS_Strict`。
- **L394 EN**: Declares TableGen def `OMP_NUMTASKS_Unknown`.
  **L394 CN**: 声明 TableGen def `OMP_NUMTASKS_Unknown`。
- **L395 EN**: Declares TableGen def `OMPC_NumTasks`.
  **L395 CN**: 声明 TableGen def `OMPC_NumTasks`。
- **L396 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L396 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L397 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L397 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L398 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L398 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L399 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L399 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L400 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OMP_NUMTASKS_Strict,`.
  **L400 CN**: 继续一个多行参数列表、初始化器或聚合项：`OMP_NUMTASKS_Strict,`。
- **L401 EN**: Continues the surrounding expression or declaration: `OMP_NUMTASKS_Unknown`.
  **L401 CN**: 继续构造周围的表达式或声明：`OMP_NUMTASKS_Unknown`。
- **L402 EN**: Executes a standalone statement or declaration: `];`.
  **L402 CN**: 执行一条独立语句或声明：`];`。
- **L403 EN**: Closes the current lexical scope or compound statement.
  **L403 CN**: 结束当前词法作用域或复合语句块。
- **L404 EN**: Declares TableGen def `OMPC_NumTeams`.
  **L404 CN**: 声明 TableGen def `OMPC_NumTeams`。
- **L405 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L405 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L406 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L406 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L407 EN**: Closes the current lexical scope or compound statement.
  **L407 CN**: 结束当前词法作用域或复合语句块。
- **L408 EN**: Declares TableGen def `OMP_NUMTHREADS_Strict`.
  **L408 CN**: 声明 TableGen def `OMP_NUMTHREADS_Strict`。
- **L409 EN**: Declares TableGen def `OMP_NUMTHREADS_Unknown`.
  **L409 CN**: 声明 TableGen def `OMP_NUMTHREADS_Unknown`。
- **L410 EN**: Declares TableGen def `OMPC_NumThreads`.
  **L410 CN**: 声明 TableGen def `OMPC_NumThreads`。
- **L411 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L411 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L412 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L412 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L413 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L413 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L414 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L414 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L415 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OMP_NUMTHREADS_Strict,`.
  **L415 CN**: 继续一个多行参数列表、初始化器或聚合项：`OMP_NUMTHREADS_Strict,`。
- **L416 EN**: Continues the surrounding expression or declaration: `OMP_NUMTHREADS_Unknown`.
  **L416 CN**: 继续构造周围的表达式或声明：`OMP_NUMTHREADS_Unknown`。

### Lines 417-448

````tablegen
  ];
}
def OMPC_OMPX_Attribute : Clause<[Spelling<"ompx_attribute">]> {
  let clangClass = "OMPXAttributeClause";
}
def OMPC_OMPX_Bare : Clause<[Spelling<"ompx_bare">]> {
  let clangClass = "OMPXBareClause";
}
def OMPC_OMPX_DynCGroupMem : Clause<[Spelling<"ompx_dyn_cgroup_mem">]> {
  let clangClass = "OMPXDynCGroupMemClause";
  let flangClass = "ScalarIntExpr";
}
def OMP_ORDER_concurrent : EnumVal<"concurrent",1,1> {}
def OMP_ORDER_unknown : EnumVal<"unknown",2,0> { let isDefault = 1; }
def OMPC_Order : Clause<[Spelling<"order">]> {
  let clangClass = "OMPOrderClause";
  let flangClass = "OmpOrderClause";
  let enumClauseValue = "OrderKind";
  let allowedClauseValues = [
    OMP_ORDER_unknown,
    OMP_ORDER_concurrent
  ];
}
def OMPC_Ordered : Clause<[Spelling<"ordered">]> {
  let clangClass = "OMPOrderedClause";
  let flangClass = "ScalarIntConstantExpr";
  let isValueOptional = true;
}
def OMPC_Otherwise : Clause<[Spelling<"otherwise">]> {
  let flangClass = "OmpOtherwiseClause";
  let isValueOptional = true;
}
````
- **L417 EN**: Executes a standalone statement or declaration: `];`.
  **L417 CN**: 执行一条独立语句或声明：`];`。
- **L418 EN**: Closes the current lexical scope or compound statement.
  **L418 CN**: 结束当前词法作用域或复合语句块。
- **L419 EN**: Declares TableGen def `OMPC_OMPX_Attribute`.
  **L419 CN**: 声明 TableGen def `OMPC_OMPX_Attribute`。
- **L420 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L420 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L421 EN**: Closes the current lexical scope or compound statement.
  **L421 CN**: 结束当前词法作用域或复合语句块。
- **L422 EN**: Declares TableGen def `OMPC_OMPX_Bare`.
  **L422 CN**: 声明 TableGen def `OMPC_OMPX_Bare`。
- **L423 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L423 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L424 EN**: Closes the current lexical scope or compound statement.
  **L424 CN**: 结束当前词法作用域或复合语句块。
- **L425 EN**: Declares TableGen def `OMPC_OMPX_DynCGroupMem`.
  **L425 CN**: 声明 TableGen def `OMPC_OMPX_DynCGroupMem`。
- **L426 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L426 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L427 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L427 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L428 EN**: Closes the current lexical scope or compound statement.
  **L428 CN**: 结束当前词法作用域或复合语句块。
- **L429 EN**: Declares TableGen def `OMP_ORDER_concurrent`.
  **L429 CN**: 声明 TableGen def `OMP_ORDER_concurrent`。
- **L430 EN**: Declares TableGen def `OMP_ORDER_unknown`.
  **L430 CN**: 声明 TableGen def `OMP_ORDER_unknown`。
- **L431 EN**: Declares TableGen def `OMPC_Order`.
  **L431 CN**: 声明 TableGen def `OMPC_Order`。
- **L432 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L432 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L433 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L433 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L434 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L434 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L435 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L435 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L436 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OMP_ORDER_unknown,`.
  **L436 CN**: 继续一个多行参数列表、初始化器或聚合项：`OMP_ORDER_unknown,`。
- **L437 EN**: Continues the surrounding expression or declaration: `OMP_ORDER_concurrent`.
  **L437 CN**: 继续构造周围的表达式或声明：`OMP_ORDER_concurrent`。
- **L438 EN**: Executes a standalone statement or declaration: `];`.
  **L438 CN**: 执行一条独立语句或声明：`];`。
- **L439 EN**: Closes the current lexical scope or compound statement.
  **L439 CN**: 结束当前词法作用域或复合语句块。
- **L440 EN**: Declares TableGen def `OMPC_Ordered`.
  **L440 CN**: 声明 TableGen def `OMPC_Ordered`。
- **L441 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L441 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L442 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L442 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L443 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L443 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L444 EN**: Closes the current lexical scope or compound statement.
  **L444 CN**: 结束当前词法作用域或复合语句块。
- **L445 EN**: Declares TableGen def `OMPC_Otherwise`.
  **L445 CN**: 声明 TableGen def `OMPC_Otherwise`。
- **L446 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L446 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L447 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L447 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L448 EN**: Closes the current lexical scope or compound statement.
  **L448 CN**: 结束当前词法作用域或复合语句块。

### Lines 449-480

````tablegen
def OMPC_Partial: Clause<[Spelling<"partial">]> {
  let clangClass = "OMPPartialClause";
  let flangClass = "ScalarIntConstantExpr";
  let isValueOptional = true;
}
def OMPC_Permutation: Clause<[Spelling<"permutation">]> {
  let clangClass = "OMPPermutationClause";
  let flangClass = "ScalarIntConstantExpr";
  let isValueList = true;
}
def OMPC_Priority : Clause<[Spelling<"priority">]> {
  let clangClass = "OMPPriorityClause";
  let flangClass = "ScalarIntExpr";
}
def OMPC_Private : Clause<[Spelling<"private">]> {
  let clangClass = "OMPPrivateClause";
  let flangClass = "OmpObjectList";
}
def OMP_PROC_BIND_master : EnumVal<"master",2,1> {}
def OMP_PROC_BIND_close : EnumVal<"close",3,1> {}
def OMP_PROC_BIND_spread : EnumVal<"spread",4,1> {}
def OMP_PROC_BIND_primary : EnumVal<"primary",5,1> {}
def OMP_PROC_BIND_default : EnumVal<"default",6,0> {}
def OMP_PROC_BIND_unknown : EnumVal<"unknown",7,0> { let isDefault = true; }
def OMPC_ProcBind : Clause<[Spelling<"proc_bind">]> {
  let clangClass = "OMPProcBindClause";
  let flangClass = "OmpProcBindClause";
  let enumClauseValue = "ProcBindKind";
  let allowedClauseValues = [
    OMP_PROC_BIND_primary,
    OMP_PROC_BIND_master,
    OMP_PROC_BIND_close,
````
- **L449 EN**: Declares TableGen def `OMPC_Partial`.
  **L449 CN**: 声明 TableGen def `OMPC_Partial`。
- **L450 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L450 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L451 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L451 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L452 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L452 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L453 EN**: Closes the current lexical scope or compound statement.
  **L453 CN**: 结束当前词法作用域或复合语句块。
- **L454 EN**: Declares TableGen def `OMPC_Permutation`.
  **L454 CN**: 声明 TableGen def `OMPC_Permutation`。
- **L455 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L455 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L456 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L456 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L457 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L457 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L458 EN**: Closes the current lexical scope or compound statement.
  **L458 CN**: 结束当前词法作用域或复合语句块。
- **L459 EN**: Declares TableGen def `OMPC_Priority`.
  **L459 CN**: 声明 TableGen def `OMPC_Priority`。
- **L460 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L460 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L461 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L461 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L462 EN**: Closes the current lexical scope or compound statement.
  **L462 CN**: 结束当前词法作用域或复合语句块。
- **L463 EN**: Declares TableGen def `OMPC_Private`.
  **L463 CN**: 声明 TableGen def `OMPC_Private`。
- **L464 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L464 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L465 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L465 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L466 EN**: Closes the current lexical scope or compound statement.
  **L466 CN**: 结束当前词法作用域或复合语句块。
- **L467 EN**: Declares TableGen def `OMP_PROC_BIND_master`.
  **L467 CN**: 声明 TableGen def `OMP_PROC_BIND_master`。
- **L468 EN**: Declares TableGen def `OMP_PROC_BIND_close`.
  **L468 CN**: 声明 TableGen def `OMP_PROC_BIND_close`。
- **L469 EN**: Declares TableGen def `OMP_PROC_BIND_spread`.
  **L469 CN**: 声明 TableGen def `OMP_PROC_BIND_spread`。
- **L470 EN**: Declares TableGen def `OMP_PROC_BIND_primary`.
  **L470 CN**: 声明 TableGen def `OMP_PROC_BIND_primary`。
- **L471 EN**: Declares TableGen def `OMP_PROC_BIND_default`.
  **L471 CN**: 声明 TableGen def `OMP_PROC_BIND_default`。
- **L472 EN**: Declares TableGen def `OMP_PROC_BIND_unknown`.
  **L472 CN**: 声明 TableGen def `OMP_PROC_BIND_unknown`。
- **L473 EN**: Declares TableGen def `OMPC_ProcBind`.
  **L473 CN**: 声明 TableGen def `OMPC_ProcBind`。
- **L474 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L474 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L475 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L475 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L476 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L476 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L477 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L477 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L478 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OMP_PROC_BIND_primary,`.
  **L478 CN**: 继续一个多行参数列表、初始化器或聚合项：`OMP_PROC_BIND_primary,`。
- **L479 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OMP_PROC_BIND_master,`.
  **L479 CN**: 继续一个多行参数列表、初始化器或聚合项：`OMP_PROC_BIND_master,`。
- **L480 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OMP_PROC_BIND_close,`.
  **L480 CN**: 继续一个多行参数列表、初始化器或聚合项：`OMP_PROC_BIND_close,`。

### Lines 481-512

````tablegen
    OMP_PROC_BIND_spread,
    OMP_PROC_BIND_default,
    OMP_PROC_BIND_unknown
  ];
}
def OMPC_Read : Clause<[Spelling<"read">]> {
  let clangClass = "OMPReadClause";
}
def OMPC_Reduction : Clause<[Spelling<"reduction">]> {
  let clangClass = "OMPReductionClause";
  let flangClass = "OmpReductionClause";
}
def OMPC_Relaxed : Clause<[Spelling<"relaxed">]> {
  let clangClass = "OMPRelaxedClause";
}
def OMPC_Release : Clause<[Spelling<"release">]> {
  let clangClass = "OMPReleaseClause";
}
def OMPC_Replayable : Clause<[Spelling<"replayable">]> {
  let flangClass = "OmpReplayableClause";
  let isValueOptional = true;
}
def OMPC_ReverseOffload : Clause<[Spelling<"reverse_offload">]> {
  let clangClass = "OMPReverseOffloadClause";
  let flangClass = "OmpReverseOffloadClause";
  let isValueOptional = true;
}
def OMPC_SafeLen : Clause<[Spelling<"safelen">]> {
  let clangClass = "OMPSafelenClause";
  let flangClass = "ScalarIntConstantExpr";
}
def OMPC_Safesync : Clause<[Spelling<"safesync">]> {
````
- **L481 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OMP_PROC_BIND_spread,`.
  **L481 CN**: 继续一个多行参数列表、初始化器或聚合项：`OMP_PROC_BIND_spread,`。
- **L482 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OMP_PROC_BIND_default,`.
  **L482 CN**: 继续一个多行参数列表、初始化器或聚合项：`OMP_PROC_BIND_default,`。
- **L483 EN**: Continues the surrounding expression or declaration: `OMP_PROC_BIND_unknown`.
  **L483 CN**: 继续构造周围的表达式或声明：`OMP_PROC_BIND_unknown`。
- **L484 EN**: Executes a standalone statement or declaration: `];`.
  **L484 CN**: 执行一条独立语句或声明：`];`。
- **L485 EN**: Closes the current lexical scope or compound statement.
  **L485 CN**: 结束当前词法作用域或复合语句块。
- **L486 EN**: Declares TableGen def `OMPC_Read`.
  **L486 CN**: 声明 TableGen def `OMPC_Read`。
- **L487 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L487 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L488 EN**: Closes the current lexical scope or compound statement.
  **L488 CN**: 结束当前词法作用域或复合语句块。
- **L489 EN**: Declares TableGen def `OMPC_Reduction`.
  **L489 CN**: 声明 TableGen def `OMPC_Reduction`。
- **L490 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L490 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L491 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L491 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L492 EN**: Closes the current lexical scope or compound statement.
  **L492 CN**: 结束当前词法作用域或复合语句块。
- **L493 EN**: Declares TableGen def `OMPC_Relaxed`.
  **L493 CN**: 声明 TableGen def `OMPC_Relaxed`。
- **L494 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L494 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L495 EN**: Closes the current lexical scope or compound statement.
  **L495 CN**: 结束当前词法作用域或复合语句块。
- **L496 EN**: Declares TableGen def `OMPC_Release`.
  **L496 CN**: 声明 TableGen def `OMPC_Release`。
- **L497 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L497 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L498 EN**: Closes the current lexical scope or compound statement.
  **L498 CN**: 结束当前词法作用域或复合语句块。
- **L499 EN**: Declares TableGen def `OMPC_Replayable`.
  **L499 CN**: 声明 TableGen def `OMPC_Replayable`。
- **L500 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L500 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L501 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L501 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L502 EN**: Closes the current lexical scope or compound statement.
  **L502 CN**: 结束当前词法作用域或复合语句块。
- **L503 EN**: Declares TableGen def `OMPC_ReverseOffload`.
  **L503 CN**: 声明 TableGen def `OMPC_ReverseOffload`。
- **L504 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L504 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L505 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L505 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L506 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L506 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L507 EN**: Closes the current lexical scope or compound statement.
  **L507 CN**: 结束当前词法作用域或复合语句块。
- **L508 EN**: Declares TableGen def `OMPC_SafeLen`.
  **L508 CN**: 声明 TableGen def `OMPC_SafeLen`。
- **L509 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L509 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L510 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L510 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L511 EN**: Closes the current lexical scope or compound statement.
  **L511 CN**: 结束当前词法作用域或复合语句块。
- **L512 EN**: Declares TableGen def `OMPC_Safesync`.
  **L512 CN**: 声明 TableGen def `OMPC_Safesync`。

### Lines 513-544

````tablegen
  let isValueOptional = true;
}
def OMP_SCHEDULE_Static : EnumVal<"static", 2, 1> {}
def OMP_SCHEDULE_Dynamic : EnumVal<"dynamic", 3, 1> {}
def OMP_SCHEDULE_Guided : EnumVal<"guided", 4, 1> {}
def OMP_SCHEDULE_Auto : EnumVal<"auto", 5, 1> {}
def OMP_SCHEDULE_Runtime : EnumVal<"runtime", 6, 1> {}
def OMP_SCHEDULE_Distribute : EnumVal<"distribute", 7, 1> {}
def OMP_SCHEDULE_Default : EnumVal<"default", 8, 0> { let isDefault = 1; }
def OMPC_Schedule : Clause<[Spelling<"schedule">]> {
  let clangClass = "OMPScheduleClause";
  let flangClass = "OmpScheduleClause";
  let enumClauseValue = "ScheduleKind";
  let allowedClauseValues = [
    OMP_SCHEDULE_Static,
    OMP_SCHEDULE_Dynamic,
    OMP_SCHEDULE_Guided,
    OMP_SCHEDULE_Auto,
    OMP_SCHEDULE_Runtime,
    OMP_SCHEDULE_Distribute,
    OMP_SCHEDULE_Default
  ];
}
def OMPC_SelfMaps : Clause<[Spelling<"self_maps">]> {
  let clangClass = "OMPSelfMapsClause";
  let flangClass = "OmpSelfMapsClause";
  let isValueOptional = true;
}
def OMPC_SeqCst : Clause<[Spelling<"seq_cst">]> {
  let clangClass = "OMPSeqCstClause";
}
def OMPC_Severity : Clause<[Spelling<"severity">]> {
````
- **L513 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L513 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L514 EN**: Closes the current lexical scope or compound statement.
  **L514 CN**: 结束当前词法作用域或复合语句块。
- **L515 EN**: Declares TableGen def `OMP_SCHEDULE_Static`.
  **L515 CN**: 声明 TableGen def `OMP_SCHEDULE_Static`。
- **L516 EN**: Declares TableGen def `OMP_SCHEDULE_Dynamic`.
  **L516 CN**: 声明 TableGen def `OMP_SCHEDULE_Dynamic`。
- **L517 EN**: Declares TableGen def `OMP_SCHEDULE_Guided`.
  **L517 CN**: 声明 TableGen def `OMP_SCHEDULE_Guided`。
- **L518 EN**: Declares TableGen def `OMP_SCHEDULE_Auto`.
  **L518 CN**: 声明 TableGen def `OMP_SCHEDULE_Auto`。
- **L519 EN**: Declares TableGen def `OMP_SCHEDULE_Runtime`.
  **L519 CN**: 声明 TableGen def `OMP_SCHEDULE_Runtime`。
- **L520 EN**: Declares TableGen def `OMP_SCHEDULE_Distribute`.
  **L520 CN**: 声明 TableGen def `OMP_SCHEDULE_Distribute`。
- **L521 EN**: Declares TableGen def `OMP_SCHEDULE_Default`.
  **L521 CN**: 声明 TableGen def `OMP_SCHEDULE_Default`。
- **L522 EN**: Declares TableGen def `OMPC_Schedule`.
  **L522 CN**: 声明 TableGen def `OMPC_Schedule`。
- **L523 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L523 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L524 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L524 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L525 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L525 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L526 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L526 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L527 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OMP_SCHEDULE_Static,`.
  **L527 CN**: 继续一个多行参数列表、初始化器或聚合项：`OMP_SCHEDULE_Static,`。
- **L528 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OMP_SCHEDULE_Dynamic,`.
  **L528 CN**: 继续一个多行参数列表、初始化器或聚合项：`OMP_SCHEDULE_Dynamic,`。
- **L529 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OMP_SCHEDULE_Guided,`.
  **L529 CN**: 继续一个多行参数列表、初始化器或聚合项：`OMP_SCHEDULE_Guided,`。
- **L530 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OMP_SCHEDULE_Auto,`.
  **L530 CN**: 继续一个多行参数列表、初始化器或聚合项：`OMP_SCHEDULE_Auto,`。
- **L531 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OMP_SCHEDULE_Runtime,`.
  **L531 CN**: 继续一个多行参数列表、初始化器或聚合项：`OMP_SCHEDULE_Runtime,`。
- **L532 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OMP_SCHEDULE_Distribute,`.
  **L532 CN**: 继续一个多行参数列表、初始化器或聚合项：`OMP_SCHEDULE_Distribute,`。
- **L533 EN**: Continues the surrounding expression or declaration: `OMP_SCHEDULE_Default`.
  **L533 CN**: 继续构造周围的表达式或声明：`OMP_SCHEDULE_Default`。
- **L534 EN**: Executes a standalone statement or declaration: `];`.
  **L534 CN**: 执行一条独立语句或声明：`];`。
- **L535 EN**: Closes the current lexical scope or compound statement.
  **L535 CN**: 结束当前词法作用域或复合语句块。
- **L536 EN**: Declares TableGen def `OMPC_SelfMaps`.
  **L536 CN**: 声明 TableGen def `OMPC_SelfMaps`。
- **L537 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L537 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L538 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L538 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L539 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L539 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L540 EN**: Closes the current lexical scope or compound statement.
  **L540 CN**: 结束当前词法作用域或复合语句块。
- **L541 EN**: Declares TableGen def `OMPC_SeqCst`.
  **L541 CN**: 声明 TableGen def `OMPC_SeqCst`。
- **L542 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L542 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L543 EN**: Closes the current lexical scope or compound statement.
  **L543 CN**: 结束当前词法作用域或复合语句块。
- **L544 EN**: Declares TableGen def `OMPC_Severity`.
  **L544 CN**: 声明 TableGen def `OMPC_Severity`。

### Lines 545-576

````tablegen
  let clangClass = "OMPSeverityClause";
  let flangClass = "OmpSeverityClause";
}
def OMPC_Shared : Clause<[Spelling<"shared">]> {
  let clangClass = "OMPSharedClause";
  let flangClass = "OmpObjectList";
}
def OMPC_Simd : Clause<[Spelling<"simd">]> {
  let clangClass = "OMPSIMDClause";
}
def OMPC_SimdLen : Clause<[Spelling<"simdlen">]> {
  let clangClass = "OMPSimdlenClause";
  let flangClass = "ScalarIntConstantExpr";
}
def OMPC_Sizes: Clause<[Spelling<"sizes">]> {
  let clangClass = "OMPSizesClause";
  let flangClass = "ScalarIntExpr";
  let isValueList = true;
}
def OMPC_TaskReduction : Clause<[Spelling<"task_reduction">]> {
  let clangClass = "OMPTaskReductionClause";
  let flangClass = "OmpTaskReductionClause";
}
def OMPC_ThreadLimit : Clause<[Spelling<"thread_limit">]> {
  let clangClass = "OMPThreadLimitClause";
  let flangClass = "OmpThreadLimitClause";
}
def OMPC_ThreadPrivate : Clause<[Spelling<"threadprivate">]> {
  let isImplicit = true;
}
def OMPC_Threads : Clause<[Spelling<"threads">]> {
  let clangClass = "OMPThreadsClause";
````
- **L545 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L545 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L546 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L546 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L547 EN**: Closes the current lexical scope or compound statement.
  **L547 CN**: 结束当前词法作用域或复合语句块。
- **L548 EN**: Declares TableGen def `OMPC_Shared`.
  **L548 CN**: 声明 TableGen def `OMPC_Shared`。
- **L549 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L549 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L550 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L550 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L551 EN**: Closes the current lexical scope or compound statement.
  **L551 CN**: 结束当前词法作用域或复合语句块。
- **L552 EN**: Declares TableGen def `OMPC_Simd`.
  **L552 CN**: 声明 TableGen def `OMPC_Simd`。
- **L553 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L553 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L554 EN**: Closes the current lexical scope or compound statement.
  **L554 CN**: 结束当前词法作用域或复合语句块。
- **L555 EN**: Declares TableGen def `OMPC_SimdLen`.
  **L555 CN**: 声明 TableGen def `OMPC_SimdLen`。
- **L556 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L556 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L557 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L557 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L558 EN**: Closes the current lexical scope or compound statement.
  **L558 CN**: 结束当前词法作用域或复合语句块。
- **L559 EN**: Declares TableGen def `OMPC_Sizes`.
  **L559 CN**: 声明 TableGen def `OMPC_Sizes`。
- **L560 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L560 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L561 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L561 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L562 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L562 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L563 EN**: Closes the current lexical scope or compound statement.
  **L563 CN**: 结束当前词法作用域或复合语句块。
- **L564 EN**: Declares TableGen def `OMPC_TaskReduction`.
  **L564 CN**: 声明 TableGen def `OMPC_TaskReduction`。
- **L565 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L565 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L566 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L566 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L567 EN**: Closes the current lexical scope or compound statement.
  **L567 CN**: 结束当前词法作用域或复合语句块。
- **L568 EN**: Declares TableGen def `OMPC_ThreadLimit`.
  **L568 CN**: 声明 TableGen def `OMPC_ThreadLimit`。
- **L569 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L569 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L570 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L570 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L571 EN**: Closes the current lexical scope or compound statement.
  **L571 CN**: 结束当前词法作用域或复合语句块。
- **L572 EN**: Declares TableGen def `OMPC_ThreadPrivate`.
  **L572 CN**: 声明 TableGen def `OMPC_ThreadPrivate`。
- **L573 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L573 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L574 EN**: Closes the current lexical scope or compound statement.
  **L574 CN**: 结束当前词法作用域或复合语句块。
- **L575 EN**: Declares TableGen def `OMPC_Threads`.
  **L575 CN**: 声明 TableGen def `OMPC_Threads`。
- **L576 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L576 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 577-608

````tablegen
}
def OMPC_Threadset : Clause<[Spelling<"threadset">]> {
  let clangClass = "OMPThreadsetClause";
  let flangClass = "OmpThreadsetClause";
}
def OMPC_To : Clause<[Spelling<"to">]> {
  let clangClass = "OMPToClause";
  let flangClass = "OmpToClause";
}
def OMPC_Transparent : Clause<[Spelling<"transparent">]> {
  let clangClass = "OMPTransparentClause";
  let flangClass = "OmpTransparentClause";
  let isValueOptional = true;
}
def OMPC_UnifiedAddress : Clause<[Spelling<"unified_address">]> {
  let clangClass = "OMPUnifiedAddressClause";
  let flangClass = "OmpUnifiedAddressClause";
  let isValueOptional = true;
}
def OMPC_UnifiedSharedMemory : Clause<[Spelling<"unified_shared_memory">]> {
  let clangClass = "OMPUnifiedSharedMemoryClause";
  let flangClass = "OmpUnifiedSharedMemoryClause";
  let isValueOptional = true;
}
def OMPC_Uniform : Clause<[Spelling<"uniform">]> {
  let flangClass = "Name";
  let isValueList = true;
}
def OMPC_Unknown : Clause<[Spelling<"unknown">]> {
  let isImplicit = true;
  let isDefault = true;
}
````
- **L577 EN**: Closes the current lexical scope or compound statement.
  **L577 CN**: 结束当前词法作用域或复合语句块。
- **L578 EN**: Declares TableGen def `OMPC_Threadset`.
  **L578 CN**: 声明 TableGen def `OMPC_Threadset`。
- **L579 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L579 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L580 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L580 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L581 EN**: Closes the current lexical scope or compound statement.
  **L581 CN**: 结束当前词法作用域或复合语句块。
- **L582 EN**: Declares TableGen def `OMPC_To`.
  **L582 CN**: 声明 TableGen def `OMPC_To`。
- **L583 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L583 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L584 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L584 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L585 EN**: Closes the current lexical scope or compound statement.
  **L585 CN**: 结束当前词法作用域或复合语句块。
- **L586 EN**: Declares TableGen def `OMPC_Transparent`.
  **L586 CN**: 声明 TableGen def `OMPC_Transparent`。
- **L587 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L587 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L588 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L588 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L589 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L589 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L590 EN**: Closes the current lexical scope or compound statement.
  **L590 CN**: 结束当前词法作用域或复合语句块。
- **L591 EN**: Declares TableGen def `OMPC_UnifiedAddress`.
  **L591 CN**: 声明 TableGen def `OMPC_UnifiedAddress`。
- **L592 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L592 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L593 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L593 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L594 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L594 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L595 EN**: Closes the current lexical scope or compound statement.
  **L595 CN**: 结束当前词法作用域或复合语句块。
- **L596 EN**: Declares TableGen def `OMPC_UnifiedSharedMemory`.
  **L596 CN**: 声明 TableGen def `OMPC_UnifiedSharedMemory`。
- **L597 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L597 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L598 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L598 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L599 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L599 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L600 EN**: Closes the current lexical scope or compound statement.
  **L600 CN**: 结束当前词法作用域或复合语句块。
- **L601 EN**: Declares TableGen def `OMPC_Uniform`.
  **L601 CN**: 声明 TableGen def `OMPC_Uniform`。
- **L602 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L602 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L603 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L603 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L604 EN**: Closes the current lexical scope or compound statement.
  **L604 CN**: 结束当前词法作用域或复合语句块。
- **L605 EN**: Declares TableGen def `OMPC_Unknown`.
  **L605 CN**: 声明 TableGen def `OMPC_Unknown`。
- **L606 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L606 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L607 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L607 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L608 EN**: Closes the current lexical scope or compound statement.
  **L608 CN**: 结束当前词法作用域或复合语句块。

### Lines 609-640

````tablegen
def OMPC_Untied : Clause<[Spelling<"untied">]> {
  let clangClass = "OMPUntiedClause";
}
def OMPC_Update : Clause<[Spelling<"update">]> {
  let clangClass = "OMPUpdateClause";
  let flangClass = "OmpUpdateClause";
  let isValueOptional = true;
}
def OMPC_Use : Clause<[Spelling<"use">]> {
  let clangClass = "OMPUseClause";
  let flangClass = "OmpUseClause";
}
def OMPC_UseDeviceAddr : Clause<[Spelling<"use_device_addr">]> {
  let clangClass = "OMPUseDeviceAddrClause";
  let flangClass = "OmpObjectList";
}
def OMPC_UseDevicePtr : Clause<[Spelling<"use_device_ptr">]> {
  let clangClass = "OMPUseDevicePtrClause";
  let flangClass = "OmpObjectList";
}
def OMPC_UsesAllocators : Clause<[Spelling<"uses_allocators">]> {
  let clangClass = "OMPUsesAllocatorsClause";
}
def OMPC_Weak : Clause<[Spelling<"weak">]> {
  let clangClass = "OMPWeakClause";
}
def OMPC_When: Clause<[Spelling<"when">]> {
  let flangClass = "OmpWhenClause";
}
def OMPC_Write : Clause<[Spelling<"write">]> {
  let clangClass = "OMPWriteClause";
}
````
- **L609 EN**: Declares TableGen def `OMPC_Untied`.
  **L609 CN**: 声明 TableGen def `OMPC_Untied`。
- **L610 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L610 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L611 EN**: Closes the current lexical scope or compound statement.
  **L611 CN**: 结束当前词法作用域或复合语句块。
- **L612 EN**: Declares TableGen def `OMPC_Update`.
  **L612 CN**: 声明 TableGen def `OMPC_Update`。
- **L613 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L613 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L614 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L614 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L615 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L615 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L616 EN**: Closes the current lexical scope or compound statement.
  **L616 CN**: 结束当前词法作用域或复合语句块。
- **L617 EN**: Declares TableGen def `OMPC_Use`.
  **L617 CN**: 声明 TableGen def `OMPC_Use`。
- **L618 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L618 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L619 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L619 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L620 EN**: Closes the current lexical scope or compound statement.
  **L620 CN**: 结束当前词法作用域或复合语句块。
- **L621 EN**: Declares TableGen def `OMPC_UseDeviceAddr`.
  **L621 CN**: 声明 TableGen def `OMPC_UseDeviceAddr`。
- **L622 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L622 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L623 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L623 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L624 EN**: Closes the current lexical scope or compound statement.
  **L624 CN**: 结束当前词法作用域或复合语句块。
- **L625 EN**: Declares TableGen def `OMPC_UseDevicePtr`.
  **L625 CN**: 声明 TableGen def `OMPC_UseDevicePtr`。
- **L626 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L626 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L627 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L627 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L628 EN**: Closes the current lexical scope or compound statement.
  **L628 CN**: 结束当前词法作用域或复合语句块。
- **L629 EN**: Declares TableGen def `OMPC_UsesAllocators`.
  **L629 CN**: 声明 TableGen def `OMPC_UsesAllocators`。
- **L630 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L630 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L631 EN**: Closes the current lexical scope or compound statement.
  **L631 CN**: 结束当前词法作用域或复合语句块。
- **L632 EN**: Declares TableGen def `OMPC_Weak`.
  **L632 CN**: 声明 TableGen def `OMPC_Weak`。
- **L633 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L633 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L634 EN**: Closes the current lexical scope or compound statement.
  **L634 CN**: 结束当前词法作用域或复合语句块。
- **L635 EN**: Declares TableGen def `OMPC_When`.
  **L635 CN**: 声明 TableGen def `OMPC_When`。
- **L636 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L636 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L637 EN**: Closes the current lexical scope or compound statement.
  **L637 CN**: 结束当前词法作用域或复合语句块。
- **L638 EN**: Declares TableGen def `OMPC_Write`.
  **L638 CN**: 声明 TableGen def `OMPC_Write`。
- **L639 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L639 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L640 EN**: Closes the current lexical scope or compound statement.
  **L640 CN**: 结束当前词法作用域或复合语句块。

### Lines 641-672

````tablegen

//===----------------------------------------------------------------------===//
// Definitions of OpenMP leaf directives
// Sorted alphabetically wrt directive spelling, except "end xyz" immediately
// follows "xyz".
//===----------------------------------------------------------------------===//

def OMP_Allocate : Directive<[Spelling<"allocate">]> {
  let allowedOnceClauses = [
    VersionedClause<OMPC_Align, 51>,
    VersionedClause<OMPC_Allocator>,
  ];
  let association = AS_None;
  let category = CA_Declarative;
}
def OMP_Allocators : Directive<[Spelling<"allocators">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Allocate>,
  ];
  let association = AS_Block;
  let category = CA_Executable;
  let languages = [L_Fortran];
}
def OMP_Assumes : Directive<[Spelling<"assumes">]> {
  let association = AS_None;
  let category = CA_Informational;
  let allowedOnceClauses = [
    VersionedClause<OMPC_Absent, 51>,
    VersionedClause<OMPC_Contains, 51>,
    VersionedClause<OMPC_Holds, 51>,
    VersionedClause<OMPC_NoOpenMP, 51>,
    VersionedClause<OMPC_NoOpenMPRoutines, 51>,
````
- **L641 EN**: Blank line separating nearby declarations or logic blocks.
  **L641 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L642 EN**: Banner comment marking a file or section boundary.
  **L642 CN**: 横幅注释，用于标记文件或章节边界。
- **L643 EN**: Comment explains nearby logic, invariants, or intent: `Definitions of OpenMP leaf directives`.
  **L643 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Definitions of OpenMP leaf directives`。
- **L644 EN**: Comment explains nearby logic, invariants, or intent: `Sorted alphabetically wrt directive spelling, except "end xyz" immediately`.
  **L644 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Sorted alphabetically wrt directive spelling, except "end xyz" immediately`。
- **L645 EN**: Comment explains nearby logic, invariants, or intent: `follows "xyz".`.
  **L645 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`follows "xyz".`。
- **L646 EN**: Banner comment marking a file or section boundary.
  **L646 CN**: 横幅注释，用于标记文件或章节边界。
- **L647 EN**: Blank line separating nearby declarations or logic blocks.
  **L647 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L648 EN**: Declares TableGen def `OMP_Allocate`.
  **L648 CN**: 声明 TableGen def `OMP_Allocate`。
- **L649 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L649 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L650 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Align, 51>,`.
  **L650 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Align, 51>,`。
- **L651 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocator>,`.
  **L651 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocator>,`。
- **L652 EN**: Executes a standalone statement or declaration: `];`.
  **L652 CN**: 执行一条独立语句或声明：`];`。
- **L653 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L653 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L654 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L654 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L655 EN**: Closes the current lexical scope or compound statement.
  **L655 CN**: 结束当前词法作用域或复合语句块。
- **L656 EN**: Declares TableGen def `OMP_Allocators`.
  **L656 CN**: 声明 TableGen def `OMP_Allocators`。
- **L657 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L657 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L658 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L658 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L659 EN**: Executes a standalone statement or declaration: `];`.
  **L659 CN**: 执行一条独立语句或声明：`];`。
- **L660 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L660 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L661 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L661 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L662 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L662 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L663 EN**: Closes the current lexical scope or compound statement.
  **L663 CN**: 结束当前词法作用域或复合语句块。
- **L664 EN**: Declares TableGen def `OMP_Assumes`.
  **L664 CN**: 声明 TableGen def `OMP_Assumes`。
- **L665 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L665 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L666 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L666 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L667 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L667 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L668 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Absent, 51>,`.
  **L668 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Absent, 51>,`。
- **L669 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Contains, 51>,`.
  **L669 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Contains, 51>,`。
- **L670 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Holds, 51>,`.
  **L670 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Holds, 51>,`。
- **L671 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoOpenMP, 51>,`.
  **L671 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoOpenMP, 51>,`。
- **L672 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoOpenMPRoutines, 51>,`.
  **L672 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoOpenMPRoutines, 51>,`。

### Lines 673-704

````tablegen
    VersionedClause<OMPC_NoParallelism, 51>,
  ];
}
def OMP_Assume : Directive<[Spelling<"assume">]> {
  let association = AS_Block;
  let category = CA_Informational;
  let allowedOnceClauses = [
    VersionedClause<OMPC_Absent, 51>,
    VersionedClause<OMPC_Contains, 51>,
    VersionedClause<OMPC_Holds, 51>,
    VersionedClause<OMPC_NoOpenMP, 51>,
    VersionedClause<OMPC_NoOpenMPRoutines, 51>,
    VersionedClause<OMPC_NoParallelism, 51>,
    VersionedClause<OMPC_NoOpenMPConstructs, 60>,
  ];
}
def OMP_Atomic : Directive<[Spelling<"atomic">]> {
  let allowedOnceClauses = [
    VersionedClause<OMPC_AcqRel, 50>,
    VersionedClause<OMPC_Acquire, 50>,
    VersionedClause<OMPC_Capture>,
    VersionedClause<OMPC_Compare, 51>,
    VersionedClause<OMPC_Fail, 51>,
    VersionedClause<OMPC_Hint, 50>,
    VersionedClause<OMPC_Read>,
    VersionedClause<OMPC_Relaxed, 50>,
    VersionedClause<OMPC_Release, 50>,
    VersionedClause<OMPC_SeqCst>,
    VersionedClause<OMPC_Update>,
    VersionedClause<OMPC_Weak, 51>,
    VersionedClause<OMPC_Write>,
  ];
````
- **L673 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoParallelism, 51>,`.
  **L673 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoParallelism, 51>,`。
- **L674 EN**: Executes a standalone statement or declaration: `];`.
  **L674 CN**: 执行一条独立语句或声明：`];`。
- **L675 EN**: Closes the current lexical scope or compound statement.
  **L675 CN**: 结束当前词法作用域或复合语句块。
- **L676 EN**: Declares TableGen def `OMP_Assume`.
  **L676 CN**: 声明 TableGen def `OMP_Assume`。
- **L677 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L677 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L678 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L678 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L679 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L679 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L680 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Absent, 51>,`.
  **L680 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Absent, 51>,`。
- **L681 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Contains, 51>,`.
  **L681 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Contains, 51>,`。
- **L682 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Holds, 51>,`.
  **L682 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Holds, 51>,`。
- **L683 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoOpenMP, 51>,`.
  **L683 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoOpenMP, 51>,`。
- **L684 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoOpenMPRoutines, 51>,`.
  **L684 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoOpenMPRoutines, 51>,`。
- **L685 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoParallelism, 51>,`.
  **L685 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoParallelism, 51>,`。
- **L686 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoOpenMPConstructs, 60>,`.
  **L686 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoOpenMPConstructs, 60>,`。
- **L687 EN**: Executes a standalone statement or declaration: `];`.
  **L687 CN**: 执行一条独立语句或声明：`];`。
- **L688 EN**: Closes the current lexical scope or compound statement.
  **L688 CN**: 结束当前词法作用域或复合语句块。
- **L689 EN**: Declares TableGen def `OMP_Atomic`.
  **L689 CN**: 声明 TableGen def `OMP_Atomic`。
- **L690 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L690 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L691 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_AcqRel, 50>,`.
  **L691 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_AcqRel, 50>,`。
- **L692 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Acquire, 50>,`.
  **L692 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Acquire, 50>,`。
- **L693 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Capture>,`.
  **L693 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Capture>,`。
- **L694 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Compare, 51>,`.
  **L694 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Compare, 51>,`。
- **L695 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Fail, 51>,`.
  **L695 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Fail, 51>,`。
- **L696 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Hint, 50>,`.
  **L696 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Hint, 50>,`。
- **L697 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Read>,`.
  **L697 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Read>,`。
- **L698 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Relaxed, 50>,`.
  **L698 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Relaxed, 50>,`。
- **L699 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Release, 50>,`.
  **L699 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Release, 50>,`。
- **L700 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SeqCst>,`.
  **L700 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SeqCst>,`。
- **L701 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Update>,`.
  **L701 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Update>,`。
- **L702 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Weak, 51>,`.
  **L702 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Weak, 51>,`。
- **L703 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Write>,`.
  **L703 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Write>,`。
- **L704 EN**: Executes a standalone statement or declaration: `];`.
  **L704 CN**: 执行一条独立语句或声明：`];`。

### Lines 705-736

````tablegen
  let association = AS_Block;
  let category = CA_Executable;
}
def OMP_Barrier : Directive<[Spelling<"barrier">]> {
  let association = AS_None;
  let category = CA_Executable;
}
def OMP_BeginAssumes : Directive<[Spelling<"begin assumes">]> {
  let association = AS_Delimited;
  let category = CA_Informational;
  let allowedOnceClauses = [
    VersionedClause<OMPC_Absent, 51>,
    VersionedClause<OMPC_Contains, 51>,
    VersionedClause<OMPC_Holds, 51>,
    VersionedClause<OMPC_NoOpenMP, 51>,
    VersionedClause<OMPC_NoOpenMPRoutines, 51>,
    VersionedClause<OMPC_NoParallelism, 51>,
  ];
  let languages = [L_C];
}
def OMP_EndAssumes : Directive<[Spelling<"end assumes">]> {
  let association = AS_Delimited;
  let category = OMP_BeginAssumes.category;
  let languages = OMP_BeginAssumes.languages;
}
def OMP_BeginDeclareTarget : Directive<[Spelling<"begin declare target", 1, 52>,
                                        Spelling<"begin declare_target", 60>]> {
  let allowedClauses = [
    VersionedClause<OMPC_DeviceType>,
    VersionedClause<OMPC_Indirect, 51>,
    VersionedClause<OMPC_Link>,
    VersionedClause<OMPC_To>,
````
- **L705 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L705 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L706 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L706 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L707 EN**: Closes the current lexical scope or compound statement.
  **L707 CN**: 结束当前词法作用域或复合语句块。
- **L708 EN**: Declares TableGen def `OMP_Barrier`.
  **L708 CN**: 声明 TableGen def `OMP_Barrier`。
- **L709 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L709 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L710 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L710 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L711 EN**: Closes the current lexical scope or compound statement.
  **L711 CN**: 结束当前词法作用域或复合语句块。
- **L712 EN**: Declares TableGen def `OMP_BeginAssumes`.
  **L712 CN**: 声明 TableGen def `OMP_BeginAssumes`。
- **L713 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L713 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L714 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L714 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L715 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L715 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L716 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Absent, 51>,`.
  **L716 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Absent, 51>,`。
- **L717 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Contains, 51>,`.
  **L717 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Contains, 51>,`。
- **L718 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Holds, 51>,`.
  **L718 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Holds, 51>,`。
- **L719 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoOpenMP, 51>,`.
  **L719 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoOpenMP, 51>,`。
- **L720 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoOpenMPRoutines, 51>,`.
  **L720 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoOpenMPRoutines, 51>,`。
- **L721 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoParallelism, 51>,`.
  **L721 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoParallelism, 51>,`。
- **L722 EN**: Executes a standalone statement or declaration: `];`.
  **L722 CN**: 执行一条独立语句或声明：`];`。
- **L723 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L723 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L724 EN**: Closes the current lexical scope or compound statement.
  **L724 CN**: 结束当前词法作用域或复合语句块。
- **L725 EN**: Declares TableGen def `OMP_EndAssumes`.
  **L725 CN**: 声明 TableGen def `OMP_EndAssumes`。
- **L726 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L726 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L727 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L727 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L728 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L728 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L729 EN**: Closes the current lexical scope or compound statement.
  **L729 CN**: 结束当前词法作用域或复合语句块。
- **L730 EN**: Declares TableGen def `OMP_BeginDeclareTarget`.
  **L730 CN**: 声明 TableGen def `OMP_BeginDeclareTarget`。
- **L731 EN**: Continues the surrounding expression or declaration: `Spelling<"begin declare_target", 60>]> {`.
  **L731 CN**: 继续构造周围的表达式或声明：`Spelling<"begin declare_target", 60>]> {`。
- **L732 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L732 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L733 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DeviceType>,`.
  **L733 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DeviceType>,`。
- **L734 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Indirect, 51>,`.
  **L734 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Indirect, 51>,`。
- **L735 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Link>,`.
  **L735 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Link>,`。
- **L736 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_To>,`.
  **L736 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_To>,`。

### Lines 737-768

````tablegen
  ];
  let association = AS_Delimited;
  let category = CA_Declarative;
  let languages = [L_C];
}
def OMP_EndDeclareTarget : Directive<[Spelling<"end declare target", 1, 52>,
                                      Spelling<"end declare_target", 60>]> {
  let association = AS_Delimited;
  let category = OMP_BeginDeclareTarget.category;
  let languages = OMP_BeginDeclareTarget.languages;
}
def OMP_BeginDeclareVariant
    : Directive<[Spelling<"begin declare variant", 1, 52>,
                 Spelling<"begin declare_variant", 60>]> {
  let association = AS_Delimited;
  let category = CA_Declarative;
  let languages = [L_C];
}
def OMP_EndDeclareVariant : Directive<[Spelling<"end declare variant", 1, 52>,
                                       Spelling<"end declare_variant", 60>]> {
  let association = AS_Delimited;
  let category = OMP_BeginDeclareVariant.category;
  let languages = OMP_BeginDeclareVariant.languages;
}
def OMP_Cancel : Directive<[Spelling<"cancel">]> {
  let allowedOnceClauses = [
    VersionedClause<OMPC_CancellationConstructType>,
    VersionedClause<OMPC_If>,
  ];
  let association = AS_None;
  let category = CA_Executable;
}
````
- **L737 EN**: Executes a standalone statement or declaration: `];`.
  **L737 CN**: 执行一条独立语句或声明：`];`。
- **L738 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L738 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L739 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L739 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L740 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L740 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L741 EN**: Closes the current lexical scope or compound statement.
  **L741 CN**: 结束当前词法作用域或复合语句块。
- **L742 EN**: Declares TableGen def `OMP_EndDeclareTarget`.
  **L742 CN**: 声明 TableGen def `OMP_EndDeclareTarget`。
- **L743 EN**: Continues the surrounding expression or declaration: `Spelling<"end declare_target", 60>]> {`.
  **L743 CN**: 继续构造周围的表达式或声明：`Spelling<"end declare_target", 60>]> {`。
- **L744 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L744 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L745 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L745 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L746 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L746 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L747 EN**: Closes the current lexical scope or compound statement.
  **L747 CN**: 结束当前词法作用域或复合语句块。
- **L748 EN**: Declares TableGen def `OMP_BeginDeclareVariant`.
  **L748 CN**: 声明 TableGen def `OMP_BeginDeclareVariant`。
- **L749 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Directive<[Spelling<"begin declare variant", 1, 52>,`.
  **L749 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Directive<[Spelling<"begin declare variant", 1, 52>,`。
- **L750 EN**: Continues the surrounding expression or declaration: `Spelling<"begin declare_variant", 60>]> {`.
  **L750 CN**: 继续构造周围的表达式或声明：`Spelling<"begin declare_variant", 60>]> {`。
- **L751 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L751 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L752 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L752 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L753 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L753 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L754 EN**: Closes the current lexical scope or compound statement.
  **L754 CN**: 结束当前词法作用域或复合语句块。
- **L755 EN**: Declares TableGen def `OMP_EndDeclareVariant`.
  **L755 CN**: 声明 TableGen def `OMP_EndDeclareVariant`。
- **L756 EN**: Continues the surrounding expression or declaration: `Spelling<"end declare_variant", 60>]> {`.
  **L756 CN**: 继续构造周围的表达式或声明：`Spelling<"end declare_variant", 60>]> {`。
- **L757 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L757 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L758 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L758 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L759 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L759 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L760 EN**: Closes the current lexical scope or compound statement.
  **L760 CN**: 结束当前词法作用域或复合语句块。
- **L761 EN**: Declares TableGen def `OMP_Cancel`.
  **L761 CN**: 声明 TableGen def `OMP_Cancel`。
- **L762 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L762 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L763 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_CancellationConstructType>,`.
  **L763 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_CancellationConstructType>,`。
- **L764 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L764 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L765 EN**: Executes a standalone statement or declaration: `];`.
  **L765 CN**: 执行一条独立语句或声明：`];`。
- **L766 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L766 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L767 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L767 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L768 EN**: Closes the current lexical scope or compound statement.
  **L768 CN**: 结束当前词法作用域或复合语句块。

### Lines 769-800

````tablegen
def OMP_CancellationPoint : Directive<[Spelling<"cancellation point", 1, 52>,
                                       Spelling<"cancellation_point", 60>]> {
  let allowedOnceClauses = [
    VersionedClause<OMPC_CancellationConstructType>,
  ];
  let association = AS_None;
  let category = CA_Executable;
}
def OMP_Critical : Directive<[Spelling<"critical">]> {
  let allowedOnceClauses = [
    VersionedClause<OMPC_Hint>,
  ];
  let association = AS_Block;
  let category = CA_Executable;
}
def OMP_DeclareInduction : Directive<[Spelling<"declare_induction">]> {
  let allowedOnceClauses = [
    VersionedClause<OMPC_Collector, 60>,
    VersionedClause<OMPC_Inductor, 60>,
  ];
  let association = AS_None;
  let category = CA_Declarative;
}
def OMP_DeclareMapper : Directive<[Spelling<"declare mapper", 1, 52>,
                                   Spelling<"declare_mapper", 60>]> {
  let requiredClauses = [
    VersionedClause<OMPC_Map, 45>,
  ];
  let association = AS_None;
  let category = CA_Declarative;
}
def OMP_DeclareReduction : Directive<[Spelling<"declare reduction", 1, 52>,
````
- **L769 EN**: Declares TableGen def `OMP_CancellationPoint`.
  **L769 CN**: 声明 TableGen def `OMP_CancellationPoint`。
- **L770 EN**: Continues the surrounding expression or declaration: `Spelling<"cancellation_point", 60>]> {`.
  **L770 CN**: 继续构造周围的表达式或声明：`Spelling<"cancellation_point", 60>]> {`。
- **L771 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L771 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L772 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_CancellationConstructType>,`.
  **L772 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_CancellationConstructType>,`。
- **L773 EN**: Executes a standalone statement or declaration: `];`.
  **L773 CN**: 执行一条独立语句或声明：`];`。
- **L774 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L774 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L775 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L775 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L776 EN**: Closes the current lexical scope or compound statement.
  **L776 CN**: 结束当前词法作用域或复合语句块。
- **L777 EN**: Declares TableGen def `OMP_Critical`.
  **L777 CN**: 声明 TableGen def `OMP_Critical`。
- **L778 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L778 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L779 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Hint>,`.
  **L779 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Hint>,`。
- **L780 EN**: Executes a standalone statement or declaration: `];`.
  **L780 CN**: 执行一条独立语句或声明：`];`。
- **L781 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L781 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L782 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L782 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L783 EN**: Closes the current lexical scope or compound statement.
  **L783 CN**: 结束当前词法作用域或复合语句块。
- **L784 EN**: Declares TableGen def `OMP_DeclareInduction`.
  **L784 CN**: 声明 TableGen def `OMP_DeclareInduction`。
- **L785 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L785 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L786 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collector, 60>,`.
  **L786 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collector, 60>,`。
- **L787 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Inductor, 60>,`.
  **L787 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Inductor, 60>,`。
- **L788 EN**: Executes a standalone statement or declaration: `];`.
  **L788 CN**: 执行一条独立语句或声明：`];`。
- **L789 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L789 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L790 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L790 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L791 EN**: Closes the current lexical scope or compound statement.
  **L791 CN**: 结束当前词法作用域或复合语句块。
- **L792 EN**: Declares TableGen def `OMP_DeclareMapper`.
  **L792 CN**: 声明 TableGen def `OMP_DeclareMapper`。
- **L793 EN**: Continues the surrounding expression or declaration: `Spelling<"declare_mapper", 60>]> {`.
  **L793 CN**: 继续构造周围的表达式或声明：`Spelling<"declare_mapper", 60>]> {`。
- **L794 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L794 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L795 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Map, 45>,`.
  **L795 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Map, 45>,`。
- **L796 EN**: Executes a standalone statement or declaration: `];`.
  **L796 CN**: 执行一条独立语句或声明：`];`。
- **L797 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L797 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L798 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L798 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L799 EN**: Closes the current lexical scope or compound statement.
  **L799 CN**: 结束当前词法作用域或复合语句块。
- **L800 EN**: Declares TableGen def `OMP_DeclareReduction`.
  **L800 CN**: 声明 TableGen def `OMP_DeclareReduction`。

### Lines 801-832

````tablegen
                                      Spelling<"declare_reduction", 60>]> {
  let allowedOnceClauses = [
    VersionedClause<OMPC_Combiner, 60>,
    VersionedClause<OMPC_Initializer>,
  ];
  let association = AS_None;
  let category = CA_Declarative;
}
def OMP_DeclareSimd : Directive<[Spelling<"declare simd", 1, 52>,
                                 Spelling<"declare_simd", 60>]> {
  let allowedClauses = [
    VersionedClause<OMPC_Aligned>,
    VersionedClause<OMPC_Linear>,
    VersionedClause<OMPC_Uniform>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_SimdLen>,
  ];
  let allowedExclusiveClauses = [
    VersionedClause<OMPC_Inbranch>,
    VersionedClause<OMPC_Notinbranch>,
  ];
  let association = AS_Declaration;
  let category = CA_Declarative;
}
def OMP_DeclareTarget : Directive<[Spelling<"declare target", 1, 52>,
                                   Spelling<"declare_target", 60>]> {
  let allowedClauses = [
    VersionedClause<OMPC_Enter, 52>,
    VersionedClause<OMPC_Indirect, 51>,
    VersionedClause<OMPC_Link>,
    VersionedClause<OMPC_Local, 60>,
````
- **L801 EN**: Continues the surrounding expression or declaration: `Spelling<"declare_reduction", 60>]> {`.
  **L801 CN**: 继续构造周围的表达式或声明：`Spelling<"declare_reduction", 60>]> {`。
- **L802 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L802 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L803 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Combiner, 60>,`.
  **L803 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Combiner, 60>,`。
- **L804 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Initializer>,`.
  **L804 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Initializer>,`。
- **L805 EN**: Executes a standalone statement or declaration: `];`.
  **L805 CN**: 执行一条独立语句或声明：`];`。
- **L806 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L806 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L807 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L807 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L808 EN**: Closes the current lexical scope or compound statement.
  **L808 CN**: 结束当前词法作用域或复合语句块。
- **L809 EN**: Declares TableGen def `OMP_DeclareSimd`.
  **L809 CN**: 声明 TableGen def `OMP_DeclareSimd`。
- **L810 EN**: Continues the surrounding expression or declaration: `Spelling<"declare_simd", 60>]> {`.
  **L810 CN**: 继续构造周围的表达式或声明：`Spelling<"declare_simd", 60>]> {`。
- **L811 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L811 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L812 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Aligned>,`.
  **L812 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Aligned>,`。
- **L813 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Linear>,`.
  **L813 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Linear>,`。
- **L814 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Uniform>,`.
  **L814 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Uniform>,`。
- **L815 EN**: Executes a standalone statement or declaration: `];`.
  **L815 CN**: 执行一条独立语句或声明：`];`。
- **L816 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L816 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L817 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SimdLen>,`.
  **L817 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SimdLen>,`。
- **L818 EN**: Executes a standalone statement or declaration: `];`.
  **L818 CN**: 执行一条独立语句或声明：`];`。
- **L819 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L819 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L820 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Inbranch>,`.
  **L820 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Inbranch>,`。
- **L821 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Notinbranch>,`.
  **L821 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Notinbranch>,`。
- **L822 EN**: Executes a standalone statement or declaration: `];`.
  **L822 CN**: 执行一条独立语句或声明：`];`。
- **L823 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L823 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L824 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L824 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L825 EN**: Closes the current lexical scope or compound statement.
  **L825 CN**: 结束当前词法作用域或复合语句块。
- **L826 EN**: Declares TableGen def `OMP_DeclareTarget`.
  **L826 CN**: 声明 TableGen def `OMP_DeclareTarget`。
- **L827 EN**: Continues the surrounding expression or declaration: `Spelling<"declare_target", 60>]> {`.
  **L827 CN**: 继续构造周围的表达式或声明：`Spelling<"declare_target", 60>]> {`。
- **L828 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L828 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L829 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Enter, 52>,`.
  **L829 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Enter, 52>,`。
- **L830 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Indirect, 51>,`.
  **L830 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Indirect, 51>,`。
- **L831 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Link>,`.
  **L831 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Link>,`。
- **L832 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Local, 60>,`.
  **L832 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Local, 60>,`。

### Lines 833-864

````tablegen
    VersionedClause<OMPC_To>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_DeviceType, 50>,
  ];
  let association = AS_None;
  let category = CA_Declarative;
}
def OMP_DeclareVariant : Directive<[Spelling<"declare variant", 1, 52>,
                                    Spelling<"declare_variant", 60>]> {
  let allowedClauses = [
    VersionedClause<OMPC_AdjustArgs, 51>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_Match>,
    VersionedClause<OMPC_AppendArgs, 51>,
  ];
  let association = AS_Declaration;
  let category = CA_Declarative;
  let languages = [L_C];
}
def OMP_Depobj : Directive<[Spelling<"depobj">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Depend, 50>,
    // TODO This should ne `none` instead. Comment carried over from
    // OMPKinds.def.
    VersionedClause<OMPC_Depobj, 50>,
    VersionedClause<OMPC_Destroy, 50>,
    VersionedClause<OMPC_Init, 60>,
    VersionedClause<OMPC_Update, 50>,
  ];
  let association = AS_None;
````
- **L833 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_To>,`.
  **L833 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_To>,`。
- **L834 EN**: Executes a standalone statement or declaration: `];`.
  **L834 CN**: 执行一条独立语句或声明：`];`。
- **L835 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L835 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L836 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DeviceType, 50>,`.
  **L836 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DeviceType, 50>,`。
- **L837 EN**: Executes a standalone statement or declaration: `];`.
  **L837 CN**: 执行一条独立语句或声明：`];`。
- **L838 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L838 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L839 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L839 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L840 EN**: Closes the current lexical scope or compound statement.
  **L840 CN**: 结束当前词法作用域或复合语句块。
- **L841 EN**: Declares TableGen def `OMP_DeclareVariant`.
  **L841 CN**: 声明 TableGen def `OMP_DeclareVariant`。
- **L842 EN**: Continues the surrounding expression or declaration: `Spelling<"declare_variant", 60>]> {`.
  **L842 CN**: 继续构造周围的表达式或声明：`Spelling<"declare_variant", 60>]> {`。
- **L843 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L843 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L844 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_AdjustArgs, 51>,`.
  **L844 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_AdjustArgs, 51>,`。
- **L845 EN**: Executes a standalone statement or declaration: `];`.
  **L845 CN**: 执行一条独立语句或声明：`];`。
- **L846 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L846 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L847 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Match>,`.
  **L847 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Match>,`。
- **L848 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_AppendArgs, 51>,`.
  **L848 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_AppendArgs, 51>,`。
- **L849 EN**: Executes a standalone statement or declaration: `];`.
  **L849 CN**: 执行一条独立语句或声明：`];`。
- **L850 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L850 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L851 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L851 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L852 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L852 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L853 EN**: Closes the current lexical scope or compound statement.
  **L853 CN**: 结束当前词法作用域或复合语句块。
- **L854 EN**: Declares TableGen def `OMP_Depobj`.
  **L854 CN**: 声明 TableGen def `OMP_Depobj`。
- **L855 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L855 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L856 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Depend, 50>,`.
  **L856 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Depend, 50>,`。
- **L857 EN**: Comment records a pending task or caution: `TODO This should ne `none` instead. Comment carried over from`.
  **L857 CN**: 注释记录了待办事项或注意点：`TODO This should ne `none` instead. Comment carried over from`。
- **L858 EN**: Comment explains nearby logic, invariants, or intent: `OMPKinds.def.`.
  **L858 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`OMPKinds.def.`。
- **L859 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Depobj, 50>,`.
  **L859 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Depobj, 50>,`。
- **L860 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Destroy, 50>,`.
  **L860 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Destroy, 50>,`。
- **L861 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Init, 60>,`.
  **L861 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Init, 60>,`。
- **L862 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Update, 50>,`.
  **L862 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Update, 50>,`。
- **L863 EN**: Executes a standalone statement or declaration: `];`.
  **L863 CN**: 执行一条独立语句或声明：`];`。
- **L864 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L864 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 865-896

````tablegen
  let category = CA_Executable;
}
def OMP_dispatch : Directive<[Spelling<"dispatch">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Depend>,
    VersionedClause<OMPC_Device>,
    VersionedClause<OMPC_HasDeviceAddr, 51>,
    VersionedClause<OMPC_IsDevicePtr>,
    VersionedClause<OMPC_Nocontext>,
    VersionedClause<OMPC_Novariants>,
    VersionedClause<OMPC_NoWait>,
  ];
  let association = AS_Block;
  let category = CA_Executable;
}
def OMP_Distribute : Directive<[Spelling<"distribute">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Private>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_DistSchedule>,
    VersionedClause<OMPC_Order, 50>,
  ];
  let association = AS_LoopNest;
  let category = CA_Executable;
}
def OMP_Do : Directive<[Spelling<"do">]> {
  let allowedClauses = [
````
- **L865 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L865 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L866 EN**: Closes the current lexical scope or compound statement.
  **L866 CN**: 结束当前词法作用域或复合语句块。
- **L867 EN**: Declares TableGen def `OMP_dispatch`.
  **L867 CN**: 声明 TableGen def `OMP_dispatch`。
- **L868 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L868 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L869 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Depend>,`.
  **L869 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Depend>,`。
- **L870 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Device>,`.
  **L870 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Device>,`。
- **L871 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_HasDeviceAddr, 51>,`.
  **L871 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_HasDeviceAddr, 51>,`。
- **L872 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_IsDevicePtr>,`.
  **L872 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_IsDevicePtr>,`。
- **L873 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Nocontext>,`.
  **L873 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Nocontext>,`。
- **L874 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Novariants>,`.
  **L874 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Novariants>,`。
- **L875 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoWait>,`.
  **L875 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoWait>,`。
- **L876 EN**: Executes a standalone statement or declaration: `];`.
  **L876 CN**: 执行一条独立语句或声明：`];`。
- **L877 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L877 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L878 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L878 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L879 EN**: Closes the current lexical scope or compound statement.
  **L879 CN**: 结束当前词法作用域或复合语句块。
- **L880 EN**: Declares TableGen def `OMP_Distribute`.
  **L880 CN**: 声明 TableGen def `OMP_Distribute`。
- **L881 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L881 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L882 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L882 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L883 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L883 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L884 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L884 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L885 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L885 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L886 EN**: Executes a standalone statement or declaration: `];`.
  **L886 CN**: 执行一条独立语句或声明：`];`。
- **L887 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L887 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L888 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L888 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L889 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DistSchedule>,`.
  **L889 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DistSchedule>,`。
- **L890 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Order, 50>,`.
  **L890 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Order, 50>,`。
- **L891 EN**: Executes a standalone statement or declaration: `];`.
  **L891 CN**: 执行一条独立语句或声明：`];`。
- **L892 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L892 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L893 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L893 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L894 EN**: Closes the current lexical scope or compound statement.
  **L894 CN**: 结束当前词法作用域或复合语句块。
- **L895 EN**: Declares TableGen def `OMP_Do`.
  **L895 CN**: 声明 TableGen def `OMP_Do`。
- **L896 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L896 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 897-928

````tablegen
    VersionedClause<OMPC_Allocate, 50>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Linear>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_Reduction>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_NoWait>,
    VersionedClause<OMPC_Order, 50>,
    VersionedClause<OMPC_Ordered>,
    VersionedClause<OMPC_Schedule>,
  ];
  let association = AS_LoopNest;
  let category = CA_Executable;
  let languages = [L_Fortran];
}
def OMP_EndDo : Directive<[Spelling<"end do">]> {
  let allowedOnceClauses = [
    VersionedClause<OMPC_NoWait>,
  ];
  let leafConstructs = OMP_Do.leafConstructs;
  let association = OMP_Do.association;
  let category = OMP_Do.category;
  let languages = OMP_Do.languages;
}
def OMP_Error : Directive<[Spelling<"error">]> {
  let allowedClauses = [
    VersionedClause<OMPC_At, 51>,
    VersionedClause<OMPC_Message, 51>,
    VersionedClause<OMPC_Severity, 51>,
````
- **L897 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate, 50>,`.
  **L897 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate, 50>,`。
- **L898 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L898 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L899 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L899 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L900 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Linear>,`.
  **L900 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Linear>,`。
- **L901 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L901 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L902 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L902 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L903 EN**: Executes a standalone statement or declaration: `];`.
  **L903 CN**: 执行一条独立语句或声明：`];`。
- **L904 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L904 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L905 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L905 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L906 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoWait>,`.
  **L906 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoWait>,`。
- **L907 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Order, 50>,`.
  **L907 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Order, 50>,`。
- **L908 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Ordered>,`.
  **L908 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Ordered>,`。
- **L909 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Schedule>,`.
  **L909 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Schedule>,`。
- **L910 EN**: Executes a standalone statement or declaration: `];`.
  **L910 CN**: 执行一条独立语句或声明：`];`。
- **L911 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L911 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L912 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L912 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L913 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L913 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L914 EN**: Closes the current lexical scope or compound statement.
  **L914 CN**: 结束当前词法作用域或复合语句块。
- **L915 EN**: Declares TableGen def `OMP_EndDo`.
  **L915 CN**: 声明 TableGen def `OMP_EndDo`。
- **L916 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L916 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L917 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoWait>,`.
  **L917 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoWait>,`。
- **L918 EN**: Executes a standalone statement or declaration: `];`.
  **L918 CN**: 执行一条独立语句或声明：`];`。
- **L919 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L919 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L920 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L920 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L921 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L921 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L922 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L922 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L923 EN**: Closes the current lexical scope or compound statement.
  **L923 CN**: 结束当前词法作用域或复合语句块。
- **L924 EN**: Declares TableGen def `OMP_Error`.
  **L924 CN**: 声明 TableGen def `OMP_Error`。
- **L925 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L925 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L926 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_At, 51>,`.
  **L926 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_At, 51>,`。
- **L927 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Message, 51>,`.
  **L927 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Message, 51>,`。
- **L928 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Severity, 51>,`.
  **L928 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Severity, 51>,`。

### Lines 929-960

````tablegen
  ];
  let association = AS_None;
  let category = CA_Utility;
}
def OMP_Flatten : Directive<[Spelling<"flatten">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Apply, 60>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_Depth, 61>,
  ];
  let association = AS_LoopNest;
  let category = CA_Executable;
}
def OMP_Flush : Directive<[Spelling<"flush">]> {
  let allowedOnceClauses = [
    VersionedClause<OMPC_AcqRel, 50>,
    VersionedClause<OMPC_Acquire, 50>,
    // TODO This should ne `none` instead. Comment carried over from
    // OMPKinds.def.
    VersionedClause<OMPC_Flush>,
    VersionedClause<OMPC_Release, 50>,
    VersionedClause<OMPC_SeqCst, 51>,
  ];
  let association = AS_None;
  let category = CA_Executable;
}
def OMP_For : Directive<[Spelling<"for">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_FirstPrivate>,
````
- **L929 EN**: Executes a standalone statement or declaration: `];`.
  **L929 CN**: 执行一条独立语句或声明：`];`。
- **L930 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L930 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L931 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L931 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L932 EN**: Closes the current lexical scope or compound statement.
  **L932 CN**: 结束当前词法作用域或复合语句块。
- **L933 EN**: Declares TableGen def `OMP_Flatten`.
  **L933 CN**: 声明 TableGen def `OMP_Flatten`。
- **L934 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L934 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L935 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Apply, 60>,`.
  **L935 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Apply, 60>,`。
- **L936 EN**: Executes a standalone statement or declaration: `];`.
  **L936 CN**: 执行一条独立语句或声明：`];`。
- **L937 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L937 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L938 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Depth, 61>,`.
  **L938 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Depth, 61>,`。
- **L939 EN**: Executes a standalone statement or declaration: `];`.
  **L939 CN**: 执行一条独立语句或声明：`];`。
- **L940 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L940 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L941 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L941 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L942 EN**: Closes the current lexical scope or compound statement.
  **L942 CN**: 结束当前词法作用域或复合语句块。
- **L943 EN**: Declares TableGen def `OMP_Flush`.
  **L943 CN**: 声明 TableGen def `OMP_Flush`。
- **L944 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L944 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L945 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_AcqRel, 50>,`.
  **L945 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_AcqRel, 50>,`。
- **L946 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Acquire, 50>,`.
  **L946 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Acquire, 50>,`。
- **L947 EN**: Comment records a pending task or caution: `TODO This should ne `none` instead. Comment carried over from`.
  **L947 CN**: 注释记录了待办事项或注意点：`TODO This should ne `none` instead. Comment carried over from`。
- **L948 EN**: Comment explains nearby logic, invariants, or intent: `OMPKinds.def.`.
  **L948 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`OMPKinds.def.`。
- **L949 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Flush>,`.
  **L949 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Flush>,`。
- **L950 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Release, 50>,`.
  **L950 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Release, 50>,`。
- **L951 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SeqCst, 51>,`.
  **L951 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SeqCst, 51>,`。
- **L952 EN**: Executes a standalone statement or declaration: `];`.
  **L952 CN**: 执行一条独立语句或声明：`];`。
- **L953 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L953 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L954 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L954 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L955 EN**: Closes the current lexical scope or compound statement.
  **L955 CN**: 结束当前词法作用域或复合语句块。
- **L956 EN**: Declares TableGen def `OMP_For`.
  **L956 CN**: 声明 TableGen def `OMP_For`。
- **L957 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L957 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L958 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L958 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L959 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L959 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L960 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L960 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。

### Lines 961-992

````tablegen
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Linear>,
    VersionedClause<OMPC_NoWait>,
    VersionedClause<OMPC_Order, 50>,
    VersionedClause<OMPC_Ordered>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_Schedule>,
  ];
  let association = AS_LoopNest;
  let category = CA_Executable;
  let languages = [L_C];
}
def OMP_Groupprivate : Directive<[Spelling<"groupprivate">]> {
  let allowedOnceClauses = [
    VersionedClause<OMPC_DeviceType>,
  ];
  let association = AS_None;
  let category = CA_Declarative;
  let languages = [L_C, L_Fortran];
}
def OMP_Fuse : Directive<[Spelling<"fuse">]> {
  let allowedOnceClauses = [
    VersionedClause<OMPC_Depth, 61>,
    VersionedClause<OMPC_LoopRange, 60>,
  ];
  let association = AS_LoopSeq;
  let category = CA_Executable;
}
def OMP_Interchange : Directive<[Spelling<"interchange">]> {
  let allowedOnceClauses = [
    VersionedClause<OMPC_Permutation>,
````
- **L961 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L961 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L962 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Linear>,`.
  **L962 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Linear>,`。
- **L963 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoWait>,`.
  **L963 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoWait>,`。
- **L964 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Order, 50>,`.
  **L964 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Order, 50>,`。
- **L965 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Ordered>,`.
  **L965 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Ordered>,`。
- **L966 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L966 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L967 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L967 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L968 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Schedule>,`.
  **L968 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Schedule>,`。
- **L969 EN**: Executes a standalone statement or declaration: `];`.
  **L969 CN**: 执行一条独立语句或声明：`];`。
- **L970 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L970 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L971 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L971 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L972 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L972 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L973 EN**: Closes the current lexical scope or compound statement.
  **L973 CN**: 结束当前词法作用域或复合语句块。
- **L974 EN**: Declares TableGen def `OMP_Groupprivate`.
  **L974 CN**: 声明 TableGen def `OMP_Groupprivate`。
- **L975 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L975 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L976 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DeviceType>,`.
  **L976 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DeviceType>,`。
- **L977 EN**: Executes a standalone statement or declaration: `];`.
  **L977 CN**: 执行一条独立语句或声明：`];`。
- **L978 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L978 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L979 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L979 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L980 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L980 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L981 EN**: Closes the current lexical scope or compound statement.
  **L981 CN**: 结束当前词法作用域或复合语句块。
- **L982 EN**: Declares TableGen def `OMP_Fuse`.
  **L982 CN**: 声明 TableGen def `OMP_Fuse`。
- **L983 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L983 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L984 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Depth, 61>,`.
  **L984 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Depth, 61>,`。
- **L985 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LoopRange, 60>,`.
  **L985 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LoopRange, 60>,`。
- **L986 EN**: Executes a standalone statement or declaration: `];`.
  **L986 CN**: 执行一条独立语句或声明：`];`。
- **L987 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L987 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L988 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L988 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L989 EN**: Closes the current lexical scope or compound statement.
  **L989 CN**: 结束当前词法作用域或复合语句块。
- **L990 EN**: Declares TableGen def `OMP_Interchange`.
  **L990 CN**: 声明 TableGen def `OMP_Interchange`。
- **L991 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L991 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L992 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Permutation>,`.
  **L992 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Permutation>,`。

### Lines 993-1024

````tablegen
  ];
  let association = AS_LoopNest;
  let category = CA_Executable;
}
def OMP_interop : Directive<[Spelling<"interop">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Depend>,
    VersionedClause<OMPC_Destroy>,
    VersionedClause<OMPC_Device>,
    VersionedClause<OMPC_Init>,
    VersionedClause<OMPC_NoWait>,
    VersionedClause<OMPC_Use>,
  ];
  let association = AS_None;
  let category = CA_Executable;
}
def OMP_loop : Directive<[Spelling<"loop">]> {
  let allowedClauses = [
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_Reduction>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_Bind, 50>,
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_Order, 50>,
  ];
  let association = AS_LoopNest;
  let category = CA_Executable;
}
def OMP_masked : Directive<[Spelling<"masked">]> {
  let allowedOnceClauses = [
````
- **L993 EN**: Executes a standalone statement or declaration: `];`.
  **L993 CN**: 执行一条独立语句或声明：`];`。
- **L994 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L994 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L995 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L995 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L996 EN**: Closes the current lexical scope or compound statement.
  **L996 CN**: 结束当前词法作用域或复合语句块。
- **L997 EN**: Declares TableGen def `OMP_interop`.
  **L997 CN**: 声明 TableGen def `OMP_interop`。
- **L998 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L998 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L999 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Depend>,`.
  **L999 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Depend>,`。
- **L1000 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Destroy>,`.
  **L1000 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Destroy>,`。
- **L1001 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Device>,`.
  **L1001 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Device>,`。
- **L1002 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Init>,`.
  **L1002 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Init>,`。
- **L1003 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoWait>,`.
  **L1003 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoWait>,`。
- **L1004 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Use>,`.
  **L1004 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Use>,`。
- **L1005 EN**: Executes a standalone statement or declaration: `];`.
  **L1005 CN**: 执行一条独立语句或声明：`];`。
- **L1006 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1006 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1007 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1007 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1008 EN**: Closes the current lexical scope or compound statement.
  **L1008 CN**: 结束当前词法作用域或复合语句块。
- **L1009 EN**: Declares TableGen def `OMP_loop`.
  **L1009 CN**: 声明 TableGen def `OMP_loop`。
- **L1010 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1010 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1011 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L1011 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L1012 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L1012 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L1013 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L1013 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L1014 EN**: Executes a standalone statement or declaration: `];`.
  **L1014 CN**: 执行一条独立语句或声明：`];`。
- **L1015 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1015 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1016 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Bind, 50>,`.
  **L1016 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Bind, 50>,`。
- **L1017 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L1017 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L1018 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Order, 50>,`.
  **L1018 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Order, 50>,`。
- **L1019 EN**: Executes a standalone statement or declaration: `];`.
  **L1019 CN**: 执行一条独立语句或声明：`];`。
- **L1020 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1020 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1021 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1021 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1022 EN**: Closes the current lexical scope or compound statement.
  **L1022 CN**: 结束当前词法作用域或复合语句块。
- **L1023 EN**: Declares TableGen def `OMP_masked`.
  **L1023 CN**: 声明 TableGen def `OMP_masked`。
- **L1024 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1024 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 1025-1056

````tablegen
    VersionedClause<OMPC_Filter>,
  ];
  let association = AS_Block;
  let category = CA_Executable;
}
def OMP_Master : Directive<[Spelling<"master">]> {
  let association = AS_Block;
  let category = CA_Executable;
}
def OMP_Metadirective : Directive<[Spelling<"metadirective">]> {
  let allowedClauses = [
    VersionedClause<OMPC_When>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_Otherwise, 52>,
    VersionedClause<OMPC_Default, 50, 51>,
  ];
  let association = AS_None;
  let category = CA_Meta;
}
def OMP_Nothing : Directive<[Spelling<"nothing">]> {
  let association = AS_None;
  let category = CA_Utility;
}
def OMP_Ordered : Directive<[Spelling<"ordered">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Depend>,
    VersionedClause<OMPC_Doacross, 52>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_Simd>,
    VersionedClause<OMPC_Threads>,
````
- **L1025 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Filter>,`.
  **L1025 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Filter>,`。
- **L1026 EN**: Executes a standalone statement or declaration: `];`.
  **L1026 CN**: 执行一条独立语句或声明：`];`。
- **L1027 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1027 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1028 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1028 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1029 EN**: Closes the current lexical scope or compound statement.
  **L1029 CN**: 结束当前词法作用域或复合语句块。
- **L1030 EN**: Declares TableGen def `OMP_Master`.
  **L1030 CN**: 声明 TableGen def `OMP_Master`。
- **L1031 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1031 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1032 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1032 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1033 EN**: Closes the current lexical scope or compound statement.
  **L1033 CN**: 结束当前词法作用域或复合语句块。
- **L1034 EN**: Declares TableGen def `OMP_Metadirective`.
  **L1034 CN**: 声明 TableGen def `OMP_Metadirective`。
- **L1035 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1035 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1036 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_When>,`.
  **L1036 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_When>,`。
- **L1037 EN**: Executes a standalone statement or declaration: `];`.
  **L1037 CN**: 执行一条独立语句或声明：`];`。
- **L1038 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1038 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1039 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Otherwise, 52>,`.
  **L1039 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Otherwise, 52>,`。
- **L1040 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default, 50, 51>,`.
  **L1040 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default, 50, 51>,`。
- **L1041 EN**: Executes a standalone statement or declaration: `];`.
  **L1041 CN**: 执行一条独立语句或声明：`];`。
- **L1042 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1042 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1043 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1043 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1044 EN**: Closes the current lexical scope or compound statement.
  **L1044 CN**: 结束当前词法作用域或复合语句块。
- **L1045 EN**: Declares TableGen def `OMP_Nothing`.
  **L1045 CN**: 声明 TableGen def `OMP_Nothing`。
- **L1046 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1046 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1047 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1047 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1048 EN**: Closes the current lexical scope or compound statement.
  **L1048 CN**: 结束当前词法作用域或复合语句块。
- **L1049 EN**: Declares TableGen def `OMP_Ordered`.
  **L1049 CN**: 声明 TableGen def `OMP_Ordered`。
- **L1050 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1050 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1051 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Depend>,`.
  **L1051 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Depend>,`。
- **L1052 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Doacross, 52>,`.
  **L1052 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Doacross, 52>,`。
- **L1053 EN**: Executes a standalone statement or declaration: `];`.
  **L1053 CN**: 执行一条独立语句或声明：`];`。
- **L1054 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1054 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1055 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Simd>,`.
  **L1055 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Simd>,`。
- **L1056 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Threads>,`.
  **L1056 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Threads>,`。

### Lines 1057-1088

````tablegen
  ];
  let association = AS_None;
  // There is also a block-associated "ordered" directive.
  let category = CA_Executable;
}
def OMP_Parallel : Directive<[Spelling<"parallel">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Copyin>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_OMPX_Attribute>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_Shared>,
  ];
  let allowedOnceClauses = [VersionedClause<OMPC_Default>,
                            VersionedClause<OMPC_If>,
                            VersionedClause<OMPC_Message, 60>,
                            VersionedClause<OMPC_NumThreads>,
                            VersionedClause<OMPC_ProcBind>,
                            VersionedClause<OMPC_Severity, 60>,
  ];
  let association = AS_Block;
  let category = CA_Executable;
}
def OMP_Requires : Directive<[Spelling<"requires">]> {
  let allowedOnceClauses = [
    VersionedClause<OMPC_UnifiedAddress>,
    VersionedClause<OMPC_UnifiedSharedMemory>,
    VersionedClause<OMPC_DeviceSafesync, 60>,
    // OpenMP 5.2 Spec: If an implementation is not supporting a requirement
    // (reverse offload in this case) then it should give compile-time error
````
- **L1057 EN**: Executes a standalone statement or declaration: `];`.
  **L1057 CN**: 执行一条独立语句或声明：`];`。
- **L1058 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1058 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1059 EN**: Comment explains nearby logic, invariants, or intent: `There is also a block-associated "ordered" directive.`.
  **L1059 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`There is also a block-associated "ordered" directive.`。
- **L1060 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1060 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1061 EN**: Closes the current lexical scope or compound statement.
  **L1061 CN**: 结束当前词法作用域或复合语句块。
- **L1062 EN**: Declares TableGen def `OMP_Parallel`.
  **L1062 CN**: 声明 TableGen def `OMP_Parallel`。
- **L1063 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1063 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1064 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L1064 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L1065 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Copyin>,`.
  **L1065 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Copyin>,`。
- **L1066 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L1066 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L1067 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_Attribute>,`.
  **L1067 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_Attribute>,`。
- **L1068 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L1068 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L1069 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L1069 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L1070 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L1070 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L1071 EN**: Executes a standalone statement or declaration: `];`.
  **L1071 CN**: 执行一条独立语句或声明：`];`。
- **L1072 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1072 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1073 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L1073 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L1074 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Message, 60>,`.
  **L1074 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Message, 60>,`。
- **L1075 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumThreads>,`.
  **L1075 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumThreads>,`。
- **L1076 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ProcBind>,`.
  **L1076 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ProcBind>,`。
- **L1077 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Severity, 60>,`.
  **L1077 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Severity, 60>,`。
- **L1078 EN**: Executes a standalone statement or declaration: `];`.
  **L1078 CN**: 执行一条独立语句或声明：`];`。
- **L1079 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1079 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1080 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1080 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1081 EN**: Closes the current lexical scope or compound statement.
  **L1081 CN**: 结束当前词法作用域或复合语句块。
- **L1082 EN**: Declares TableGen def `OMP_Requires`.
  **L1082 CN**: 声明 TableGen def `OMP_Requires`。
- **L1083 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1083 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1084 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_UnifiedAddress>,`.
  **L1084 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_UnifiedAddress>,`。
- **L1085 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_UnifiedSharedMemory>,`.
  **L1085 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_UnifiedSharedMemory>,`。
- **L1086 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DeviceSafesync, 60>,`.
  **L1086 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DeviceSafesync, 60>,`。
- **L1087 EN**: Comment explains nearby logic, invariants, or intent: `OpenMP 5.2 Spec: If an implementation is not supporting a requirement`.
  **L1087 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`OpenMP 5.2 Spec: If an implementation is not supporting a requirement`。
- **L1088 EN**: Comment explains nearby logic, invariants, or intent: `(reverse offload in this case) then it should give compile-time error`.
  **L1088 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`(reverse offload in this case) then it should give compile-time error`。

### Lines 1089-1120

````tablegen
    // termination.
    // Seeting supported version for reverse_offload to a distant future version
    // 9.9 so that its partial support can be tested in the meantime.
    //
    // TODO: Correct this supprted version number whenever complete
    // implementation of reverse_offload is available.
    VersionedClause<OMPC_AtomicDefaultMemOrder>,
    VersionedClause<OMPC_DynamicAllocators>, VersionedClause<OMPC_SelfMaps>,
    VersionedClause<OMPC_ReverseOffload, 99>,
  ];
  let association = AS_None;
  let category = CA_Informational;
}
def OMP_Reverse : Directive<[Spelling<"reverse">]> {
  let association = AS_LoopNest;
  let category = CA_Executable;
}
def OMP_Scan : Directive<[Spelling<"scan">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Exclusive>,
    VersionedClause<OMPC_Inclusive>,
  ];
  let association = AS_Separating;
  let category = CA_Subsidiary;
}
def OMP_Scope : Directive<[Spelling<"scope">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Private, 51>,
    VersionedClause<OMPC_Reduction, 51>,
    VersionedClause<OMPC_FirstPrivate, 52>,
    VersionedClause<OMPC_Allocate, 52>,
  ];
````
- **L1089 EN**: Comment explains nearby logic, invariants, or intent: `termination.`.
  **L1089 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`termination.`。
- **L1090 EN**: Comment explains nearby logic, invariants, or intent: `Seeting supported version for reverse_offload to a distant future version`.
  **L1090 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Seeting supported version for reverse_offload to a distant future version`。
- **L1091 EN**: Comment explains nearby logic, invariants, or intent: `9.9 so that its partial support can be tested in the meantime.`.
  **L1091 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`9.9 so that its partial support can be tested in the meantime.`。
- **L1092 EN**: Separator comment used for visual grouping.
  **L1092 CN**: 用于视觉分组的分隔注释。
- **L1093 EN**: Comment records a pending task or caution: `TODO: Correct this supprted version number whenever complete`.
  **L1093 CN**: 注释记录了待办事项或注意点：`TODO: Correct this supprted version number whenever complete`。
- **L1094 EN**: Comment explains nearby logic, invariants, or intent: `implementation of reverse_offload is available.`.
  **L1094 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`implementation of reverse_offload is available.`。
- **L1095 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_AtomicDefaultMemOrder>,`.
  **L1095 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_AtomicDefaultMemOrder>,`。
- **L1096 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DynamicAllocators>, VersionedClause<OMPC_SelfMaps>,`.
  **L1096 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DynamicAllocators>, VersionedClause<OMPC_SelfMaps>,`。
- **L1097 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ReverseOffload, 99>,`.
  **L1097 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ReverseOffload, 99>,`。
- **L1098 EN**: Executes a standalone statement or declaration: `];`.
  **L1098 CN**: 执行一条独立语句或声明：`];`。
- **L1099 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1099 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1100 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1100 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1101 EN**: Closes the current lexical scope or compound statement.
  **L1101 CN**: 结束当前词法作用域或复合语句块。
- **L1102 EN**: Declares TableGen def `OMP_Reverse`.
  **L1102 CN**: 声明 TableGen def `OMP_Reverse`。
- **L1103 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1103 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1104 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1104 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1105 EN**: Closes the current lexical scope or compound statement.
  **L1105 CN**: 结束当前词法作用域或复合语句块。
- **L1106 EN**: Declares TableGen def `OMP_Scan`.
  **L1106 CN**: 声明 TableGen def `OMP_Scan`。
- **L1107 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1107 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1108 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Exclusive>,`.
  **L1108 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Exclusive>,`。
- **L1109 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Inclusive>,`.
  **L1109 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Inclusive>,`。
- **L1110 EN**: Executes a standalone statement or declaration: `];`.
  **L1110 CN**: 执行一条独立语句或声明：`];`。
- **L1111 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1111 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1112 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1112 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1113 EN**: Closes the current lexical scope or compound statement.
  **L1113 CN**: 结束当前词法作用域或复合语句块。
- **L1114 EN**: Declares TableGen def `OMP_Scope`.
  **L1114 CN**: 声明 TableGen def `OMP_Scope`。
- **L1115 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1115 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1116 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private, 51>,`.
  **L1116 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private, 51>,`。
- **L1117 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction, 51>,`.
  **L1117 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction, 51>,`。
- **L1118 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate, 52>,`.
  **L1118 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate, 52>,`。
- **L1119 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate, 52>,`.
  **L1119 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate, 52>,`。
- **L1120 EN**: Executes a standalone statement or declaration: `];`.
  **L1120 CN**: 执行一条独立语句或声明：`];`。

### Lines 1121-1152

````tablegen
  let allowedOnceClauses = [
    VersionedClause<OMPC_NoWait, 51>,
  ];
  let association = AS_Block;
  let category = CA_Executable;
}
def OMP_EndScope : Directive<[Spelling<"end scope">]> {
  let allowedOnceClauses = [
    VersionedClause<OMPC_NoWait>,
  ];
  let leafConstructs = OMP_Scope.leafConstructs;
  let association = OMP_Scope.association;
  let category = OMP_Scope.category;
  let languages = [L_Fortran];
}
def OMP_Section : Directive<[Spelling<"section">]> {
  let association = AS_Separating;
  let category = CA_Subsidiary;
}
def OMP_Sections : Directive<[Spelling<"sections">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_NoWait>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_Reduction>,
  ];
  let association = AS_Block;
  let category = CA_Executable;
}
def OMP_EndSections : Directive<[Spelling<"end sections">]> {
````
- **L1121 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1121 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1122 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoWait, 51>,`.
  **L1122 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoWait, 51>,`。
- **L1123 EN**: Executes a standalone statement or declaration: `];`.
  **L1123 CN**: 执行一条独立语句或声明：`];`。
- **L1124 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1124 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1125 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1125 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1126 EN**: Closes the current lexical scope or compound statement.
  **L1126 CN**: 结束当前词法作用域或复合语句块。
- **L1127 EN**: Declares TableGen def `OMP_EndScope`.
  **L1127 CN**: 声明 TableGen def `OMP_EndScope`。
- **L1128 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1128 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1129 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoWait>,`.
  **L1129 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoWait>,`。
- **L1130 EN**: Executes a standalone statement or declaration: `];`.
  **L1130 CN**: 执行一条独立语句或声明：`];`。
- **L1131 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1131 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1132 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1132 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1133 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1133 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1134 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1134 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1135 EN**: Closes the current lexical scope or compound statement.
  **L1135 CN**: 结束当前词法作用域或复合语句块。
- **L1136 EN**: Declares TableGen def `OMP_Section`.
  **L1136 CN**: 声明 TableGen def `OMP_Section`。
- **L1137 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1137 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1138 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1138 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1139 EN**: Closes the current lexical scope or compound statement.
  **L1139 CN**: 结束当前词法作用域或复合语句块。
- **L1140 EN**: Declares TableGen def `OMP_Sections`.
  **L1140 CN**: 声明 TableGen def `OMP_Sections`。
- **L1141 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1141 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1142 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L1142 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L1143 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L1143 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L1144 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L1144 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L1145 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoWait>,`.
  **L1145 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoWait>,`。
- **L1146 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L1146 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L1147 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L1147 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L1148 EN**: Executes a standalone statement or declaration: `];`.
  **L1148 CN**: 执行一条独立语句或声明：`];`。
- **L1149 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1149 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1150 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1150 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1151 EN**: Closes the current lexical scope or compound statement.
  **L1151 CN**: 结束当前词法作用域或复合语句块。
- **L1152 EN**: Declares TableGen def `OMP_EndSections`.
  **L1152 CN**: 声明 TableGen def `OMP_EndSections`。

### Lines 1153-1184

````tablegen
  let allowedOnceClauses = [
    VersionedClause<OMPC_NoWait>,
  ];
  let leafConstructs = OMP_Sections.leafConstructs;
  let association = OMP_Sections.association;
  let category = OMP_Sections.category;
  let languages = [L_Fortran];
}
def OMP_Simd : Directive<[Spelling<"simd">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Aligned>,
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Linear>,
    VersionedClause<OMPC_NonTemporal, 50>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_Reduction>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_If, 50>,
    VersionedClause<OMPC_Order, 50>,
    VersionedClause<OMPC_SafeLen>,
    VersionedClause<OMPC_SimdLen>,
  ];
  let association = AS_LoopNest;
  let category = CA_Executable;
}
def OMP_Single : Directive<[Spelling<"single">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_CopyPrivate>,
````
- **L1153 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1153 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1154 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoWait>,`.
  **L1154 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoWait>,`。
- **L1155 EN**: Executes a standalone statement or declaration: `];`.
  **L1155 CN**: 执行一条独立语句或声明：`];`。
- **L1156 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1156 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1157 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1157 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1158 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1158 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1159 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1159 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1160 EN**: Closes the current lexical scope or compound statement.
  **L1160 CN**: 结束当前词法作用域或复合语句块。
- **L1161 EN**: Declares TableGen def `OMP_Simd`.
  **L1161 CN**: 声明 TableGen def `OMP_Simd`。
- **L1162 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1162 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1163 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Aligned>,`.
  **L1163 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Aligned>,`。
- **L1164 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L1164 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L1165 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L1165 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L1166 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Linear>,`.
  **L1166 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Linear>,`。
- **L1167 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NonTemporal, 50>,`.
  **L1167 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NonTemporal, 50>,`。
- **L1168 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L1168 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L1169 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L1169 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L1170 EN**: Executes a standalone statement or declaration: `];`.
  **L1170 CN**: 执行一条独立语句或声明：`];`。
- **L1171 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1171 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1172 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L1172 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L1173 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If, 50>,`.
  **L1173 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If, 50>,`。
- **L1174 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Order, 50>,`.
  **L1174 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Order, 50>,`。
- **L1175 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SafeLen>,`.
  **L1175 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SafeLen>,`。
- **L1176 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SimdLen>,`.
  **L1176 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SimdLen>,`。
- **L1177 EN**: Executes a standalone statement or declaration: `];`.
  **L1177 CN**: 执行一条独立语句或声明：`];`。
- **L1178 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1178 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1179 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1179 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1180 EN**: Closes the current lexical scope or compound statement.
  **L1180 CN**: 结束当前词法作用域或复合语句块。
- **L1181 EN**: Declares TableGen def `OMP_Single`.
  **L1181 CN**: 声明 TableGen def `OMP_Single`。
- **L1182 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1182 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1183 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L1183 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L1184 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_CopyPrivate>,`.
  **L1184 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_CopyPrivate>,`。

### Lines 1185-1216

````tablegen
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_Private>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_NoWait>,
  ];
  let association = AS_Block;
  let category = CA_Executable;
}
def OMP_EndSingle : Directive<[Spelling<"end single">]> {
  let allowedClauses = [
    VersionedClause<OMPC_CopyPrivate>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_NoWait>,
  ];
  let leafConstructs = OMP_Single.leafConstructs;
  let association = OMP_Single.association;
  let category = OMP_Single.category;
  let languages = [L_Fortran];
}
def OMP_Target : Directive<[Spelling<"target">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Depend>,
    VersionedClause<OMPC_DynGroupprivate, 61>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_HasDeviceAddr, 51>,
    VersionedClause<OMPC_InReduction, 50>,
    VersionedClause<OMPC_IsDevicePtr>,
    VersionedClause<OMPC_Map>,
    VersionedClause<OMPC_OMPX_Attribute>,
````
- **L1185 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L1185 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L1186 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L1186 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L1187 EN**: Executes a standalone statement or declaration: `];`.
  **L1187 CN**: 执行一条独立语句或声明：`];`。
- **L1188 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1188 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1189 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoWait>,`.
  **L1189 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoWait>,`。
- **L1190 EN**: Executes a standalone statement or declaration: `];`.
  **L1190 CN**: 执行一条独立语句或声明：`];`。
- **L1191 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1191 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1192 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1192 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1193 EN**: Closes the current lexical scope or compound statement.
  **L1193 CN**: 结束当前词法作用域或复合语句块。
- **L1194 EN**: Declares TableGen def `OMP_EndSingle`.
  **L1194 CN**: 声明 TableGen def `OMP_EndSingle`。
- **L1195 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1195 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1196 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_CopyPrivate>,`.
  **L1196 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_CopyPrivate>,`。
- **L1197 EN**: Executes a standalone statement or declaration: `];`.
  **L1197 CN**: 执行一条独立语句或声明：`];`。
- **L1198 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1198 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1199 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoWait>,`.
  **L1199 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoWait>,`。
- **L1200 EN**: Executes a standalone statement or declaration: `];`.
  **L1200 CN**: 执行一条独立语句或声明：`];`。
- **L1201 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1201 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1202 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1202 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1203 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1203 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1204 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1204 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1205 EN**: Closes the current lexical scope or compound statement.
  **L1205 CN**: 结束当前词法作用域或复合语句块。
- **L1206 EN**: Declares TableGen def `OMP_Target`.
  **L1206 CN**: 声明 TableGen def `OMP_Target`。
- **L1207 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1207 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1208 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L1208 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L1209 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Depend>,`.
  **L1209 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Depend>,`。
- **L1210 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DynGroupprivate, 61>,`.
  **L1210 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DynGroupprivate, 61>,`。
- **L1211 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L1211 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L1212 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_HasDeviceAddr, 51>,`.
  **L1212 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_HasDeviceAddr, 51>,`。
- **L1213 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_InReduction, 50>,`.
  **L1213 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_InReduction, 50>,`。
- **L1214 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_IsDevicePtr>,`.
  **L1214 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_IsDevicePtr>,`。
- **L1215 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Map>,`.
  **L1215 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Map>,`。
- **L1216 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_Attribute>,`.
  **L1216 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_Attribute>,`。

### Lines 1217-1248

````tablegen
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_UsesAllocators, 50>,
    VersionedClause<OMPC_Default, 60>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_DefaultMap>,
    VersionedClause<OMPC_Device>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_NoWait>,
    VersionedClause<OMPC_OMPX_Bare>,
    VersionedClause<OMPC_OMPX_DynCGroupMem>,
    VersionedClause<OMPC_Replayable, 60>,
    VersionedClause<OMPC_ThreadLimit, 51>,
  ];
  let association = AS_Block;
  let category = CA_Executable;
}
def OMP_TargetData : Directive<[Spelling<"target data", 1, 52>,
                                Spelling<"target_data", 60>]> {
  let allowedOnceClauses = [
    VersionedClause<OMPC_Device>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_Default, 60>,
    VersionedClause<OMPC_Transparent, 60>,
  ];
  let requiredClauses = [
    VersionedClause<OMPC_Map>,
    VersionedClause<OMPC_UseDeviceAddr, 50>,
    VersionedClause<OMPC_UseDevicePtr>,
  ];
  let association = AS_Block;
  let category = CA_Executable;
````
- **L1217 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L1217 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L1218 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_UsesAllocators, 50>,`.
  **L1218 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_UsesAllocators, 50>,`。
- **L1219 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default, 60>,`.
  **L1219 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default, 60>,`。
- **L1220 EN**: Executes a standalone statement or declaration: `];`.
  **L1220 CN**: 执行一条独立语句或声明：`];`。
- **L1221 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1221 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1222 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DefaultMap>,`.
  **L1222 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DefaultMap>,`。
- **L1223 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Device>,`.
  **L1223 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Device>,`。
- **L1224 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L1224 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L1225 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoWait>,`.
  **L1225 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoWait>,`。
- **L1226 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_Bare>,`.
  **L1226 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_Bare>,`。
- **L1227 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_DynCGroupMem>,`.
  **L1227 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_DynCGroupMem>,`。
- **L1228 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Replayable, 60>,`.
  **L1228 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Replayable, 60>,`。
- **L1229 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ThreadLimit, 51>,`.
  **L1229 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ThreadLimit, 51>,`。
- **L1230 EN**: Executes a standalone statement or declaration: `];`.
  **L1230 CN**: 执行一条独立语句或声明：`];`。
- **L1231 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1231 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1232 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1232 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1233 EN**: Closes the current lexical scope or compound statement.
  **L1233 CN**: 结束当前词法作用域或复合语句块。
- **L1234 EN**: Declares TableGen def `OMP_TargetData`.
  **L1234 CN**: 声明 TableGen def `OMP_TargetData`。
- **L1235 EN**: Continues the surrounding expression or declaration: `Spelling<"target_data", 60>]> {`.
  **L1235 CN**: 继续构造周围的表达式或声明：`Spelling<"target_data", 60>]> {`。
- **L1236 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1236 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1237 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Device>,`.
  **L1237 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Device>,`。
- **L1238 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L1238 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L1239 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default, 60>,`.
  **L1239 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default, 60>,`。
- **L1240 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Transparent, 60>,`.
  **L1240 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Transparent, 60>,`。
- **L1241 EN**: Executes a standalone statement or declaration: `];`.
  **L1241 CN**: 执行一条独立语句或声明：`];`。
- **L1242 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1242 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1243 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Map>,`.
  **L1243 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Map>,`。
- **L1244 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_UseDeviceAddr, 50>,`.
  **L1244 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_UseDeviceAddr, 50>,`。
- **L1245 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_UseDevicePtr>,`.
  **L1245 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_UseDevicePtr>,`。
- **L1246 EN**: Executes a standalone statement or declaration: `];`.
  **L1246 CN**: 执行一条独立语句或声明：`];`。
- **L1247 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1247 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1248 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1248 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 1249-1280

````tablegen
}
def OMP_TargetEnterData : Directive<[Spelling<"target enter data", 1, 52>,
                                     Spelling<"target_enter_data", 60>]> {
  let allowedClauses = [
    VersionedClause<OMPC_Depend>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_Device>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_NoWait>,
    VersionedClause<OMPC_Replayable, 60>,
  ];
  let requiredClauses = [
    VersionedClause<OMPC_Map>,
  ];
  let association = AS_None;
  let category = CA_Executable;
}
def OMP_TargetExitData : Directive<[Spelling<"target exit data", 1, 52>,
                                    Spelling<"target_exit_data", 60>]> {
  let allowedClauses = [
    VersionedClause<OMPC_Depend>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_Device>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_NoWait>,
    VersionedClause<OMPC_Replayable, 60>,
  ];
  let requiredClauses = [
    VersionedClause<OMPC_Map>,
  ];
````
- **L1249 EN**: Closes the current lexical scope or compound statement.
  **L1249 CN**: 结束当前词法作用域或复合语句块。
- **L1250 EN**: Declares TableGen def `OMP_TargetEnterData`.
  **L1250 CN**: 声明 TableGen def `OMP_TargetEnterData`。
- **L1251 EN**: Continues the surrounding expression or declaration: `Spelling<"target_enter_data", 60>]> {`.
  **L1251 CN**: 继续构造周围的表达式或声明：`Spelling<"target_enter_data", 60>]> {`。
- **L1252 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1252 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1253 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Depend>,`.
  **L1253 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Depend>,`。
- **L1254 EN**: Executes a standalone statement or declaration: `];`.
  **L1254 CN**: 执行一条独立语句或声明：`];`。
- **L1255 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1255 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1256 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Device>,`.
  **L1256 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Device>,`。
- **L1257 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L1257 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L1258 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoWait>,`.
  **L1258 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoWait>,`。
- **L1259 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Replayable, 60>,`.
  **L1259 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Replayable, 60>,`。
- **L1260 EN**: Executes a standalone statement or declaration: `];`.
  **L1260 CN**: 执行一条独立语句或声明：`];`。
- **L1261 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1261 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1262 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Map>,`.
  **L1262 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Map>,`。
- **L1263 EN**: Executes a standalone statement or declaration: `];`.
  **L1263 CN**: 执行一条独立语句或声明：`];`。
- **L1264 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1264 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1265 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1265 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1266 EN**: Closes the current lexical scope or compound statement.
  **L1266 CN**: 结束当前词法作用域或复合语句块。
- **L1267 EN**: Declares TableGen def `OMP_TargetExitData`.
  **L1267 CN**: 声明 TableGen def `OMP_TargetExitData`。
- **L1268 EN**: Continues the surrounding expression or declaration: `Spelling<"target_exit_data", 60>]> {`.
  **L1268 CN**: 继续构造周围的表达式或声明：`Spelling<"target_exit_data", 60>]> {`。
- **L1269 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1269 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1270 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Depend>,`.
  **L1270 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Depend>,`。
- **L1271 EN**: Executes a standalone statement or declaration: `];`.
  **L1271 CN**: 执行一条独立语句或声明：`];`。
- **L1272 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1272 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1273 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Device>,`.
  **L1273 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Device>,`。
- **L1274 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L1274 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L1275 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoWait>,`.
  **L1275 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoWait>,`。
- **L1276 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Replayable, 60>,`.
  **L1276 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Replayable, 60>,`。
- **L1277 EN**: Executes a standalone statement or declaration: `];`.
  **L1277 CN**: 执行一条独立语句或声明：`];`。
- **L1278 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1278 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1279 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Map>,`.
  **L1279 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Map>,`。
- **L1280 EN**: Executes a standalone statement or declaration: `];`.
  **L1280 CN**: 执行一条独立语句或声明：`];`。

### Lines 1281-1312

````tablegen
  let association = AS_None;
  let category = CA_Executable;
}
def OMP_TargetUpdate : Directive<[Spelling<"target update", 1, 52>,
                                  Spelling<"target_update", 60>]> {
  let allowedClauses = [
    VersionedClause<OMPC_Depend>,
    VersionedClause<OMPC_From>,
    VersionedClause<OMPC_To>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_Device>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_NoWait>,
    VersionedClause<OMPC_Replayable, 60>,
  ];
  let association = AS_None;
  let category = CA_Executable;
}
def OMP_Task : Directive<[Spelling<"task">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Affinity, 50>,
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Depend>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_InReduction>,
    VersionedClause<OMPC_Mergeable>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_Shared>,
    VersionedClause<OMPC_Untied>,
  ];
  let allowedOnceClauses = [
````
- **L1281 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1281 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1282 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1282 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1283 EN**: Closes the current lexical scope or compound statement.
  **L1283 CN**: 结束当前词法作用域或复合语句块。
- **L1284 EN**: Declares TableGen def `OMP_TargetUpdate`.
  **L1284 CN**: 声明 TableGen def `OMP_TargetUpdate`。
- **L1285 EN**: Continues the surrounding expression or declaration: `Spelling<"target_update", 60>]> {`.
  **L1285 CN**: 继续构造周围的表达式或声明：`Spelling<"target_update", 60>]> {`。
- **L1286 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1286 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1287 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Depend>,`.
  **L1287 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Depend>,`。
- **L1288 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_From>,`.
  **L1288 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_From>,`。
- **L1289 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_To>,`.
  **L1289 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_To>,`。
- **L1290 EN**: Executes a standalone statement or declaration: `];`.
  **L1290 CN**: 执行一条独立语句或声明：`];`。
- **L1291 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1291 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1292 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Device>,`.
  **L1292 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Device>,`。
- **L1293 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L1293 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L1294 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoWait>,`.
  **L1294 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoWait>,`。
- **L1295 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Replayable, 60>,`.
  **L1295 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Replayable, 60>,`。
- **L1296 EN**: Executes a standalone statement or declaration: `];`.
  **L1296 CN**: 执行一条独立语句或声明：`];`。
- **L1297 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1297 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1298 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1298 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1299 EN**: Closes the current lexical scope or compound statement.
  **L1299 CN**: 结束当前词法作用域或复合语句块。
- **L1300 EN**: Declares TableGen def `OMP_Task`.
  **L1300 CN**: 声明 TableGen def `OMP_Task`。
- **L1301 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1301 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1302 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Affinity, 50>,`.
  **L1302 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Affinity, 50>,`。
- **L1303 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L1303 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L1304 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Depend>,`.
  **L1304 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Depend>,`。
- **L1305 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L1305 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L1306 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_InReduction>,`.
  **L1306 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_InReduction>,`。
- **L1307 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Mergeable>,`.
  **L1307 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Mergeable>,`。
- **L1308 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L1308 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L1309 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L1309 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L1310 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Untied>,`.
  **L1310 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Untied>,`。
- **L1311 EN**: Executes a standalone statement or declaration: `];`.
  **L1311 CN**: 执行一条独立语句或声明：`];`。
- **L1312 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1312 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 1313-1344

````tablegen
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_Detach, 50>,
    VersionedClause<OMPC_Final>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_Priority>,
    VersionedClause<OMPC_Threadset, 60>,
    VersionedClause<OMPC_Replayable, 60>,
    VersionedClause<OMPC_Transparent, 60>,
  ];
  let association = AS_Block;
  let category = CA_Executable;
}
def OMP_Taskgraph : Directive<[Spelling<"taskgraph">]> {
  let allowedOnceClauses = [
    VersionedClause<OMPC_GraphId, 60>,
    VersionedClause<OMPC_GraphReset, 60>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_NoGroup>,
  ];
  let association = AS_Block;
  let category = CA_Executable;
}
def OMP_TaskGroup : Directive<[Spelling<"taskgroup">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Allocate, 50>,
    VersionedClause<OMPC_TaskReduction, 50>,
  ];
  let association = AS_Block;
  let category = CA_Executable;
}
def OMP_TaskLoop : Directive<[Spelling<"taskloop">]> {
  let allowedClauses = [
````
- **L1313 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L1313 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L1314 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Detach, 50>,`.
  **L1314 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Detach, 50>,`。
- **L1315 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Final>,`.
  **L1315 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Final>,`。
- **L1316 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L1316 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L1317 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Priority>,`.
  **L1317 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Priority>,`。
- **L1318 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Threadset, 60>,`.
  **L1318 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Threadset, 60>,`。
- **L1319 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Replayable, 60>,`.
  **L1319 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Replayable, 60>,`。
- **L1320 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Transparent, 60>,`.
  **L1320 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Transparent, 60>,`。
- **L1321 EN**: Executes a standalone statement or declaration: `];`.
  **L1321 CN**: 执行一条独立语句或声明：`];`。
- **L1322 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1322 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1323 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1323 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1324 EN**: Closes the current lexical scope or compound statement.
  **L1324 CN**: 结束当前词法作用域或复合语句块。
- **L1325 EN**: Declares TableGen def `OMP_Taskgraph`.
  **L1325 CN**: 声明 TableGen def `OMP_Taskgraph`。
- **L1326 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1326 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1327 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_GraphId, 60>,`.
  **L1327 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_GraphId, 60>,`。
- **L1328 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_GraphReset, 60>,`.
  **L1328 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_GraphReset, 60>,`。
- **L1329 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L1329 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L1330 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoGroup>,`.
  **L1330 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoGroup>,`。
- **L1331 EN**: Executes a standalone statement or declaration: `];`.
  **L1331 CN**: 执行一条独立语句或声明：`];`。
- **L1332 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1332 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1333 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1333 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1334 EN**: Closes the current lexical scope or compound statement.
  **L1334 CN**: 结束当前词法作用域或复合语句块。
- **L1335 EN**: Declares TableGen def `OMP_TaskGroup`.
  **L1335 CN**: 声明 TableGen def `OMP_TaskGroup`。
- **L1336 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1336 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1337 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate, 50>,`.
  **L1337 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate, 50>,`。
- **L1338 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_TaskReduction, 50>,`.
  **L1338 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_TaskReduction, 50>,`。
- **L1339 EN**: Executes a standalone statement or declaration: `];`.
  **L1339 CN**: 执行一条独立语句或声明：`];`。
- **L1340 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1340 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1341 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1341 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1342 EN**: Closes the current lexical scope or compound statement.
  **L1342 CN**: 结束当前词法作用域或复合语句块。
- **L1343 EN**: Declares TableGen def `OMP_TaskLoop`.
  **L1343 CN**: 声明 TableGen def `OMP_TaskLoop`。
- **L1344 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1344 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 1345-1376

````tablegen
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_InReduction>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Mergeable>,
    VersionedClause<OMPC_NoGroup>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_Shared>,
    VersionedClause<OMPC_Untied>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_Final>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_Priority>,
    VersionedClause<OMPC_Threadset, 60>,
    VersionedClause<OMPC_Replayable, 60>,
    VersionedClause<OMPC_Transparent, 60>,
  ];
  let allowedExclusiveClauses = [
    VersionedClause<OMPC_GrainSize>,
    VersionedClause<OMPC_NumTasks>,
  ];
  let association = AS_LoopNest;
  let category = CA_Executable;
}
def OMP_TaskWait : Directive<[Spelling<"taskwait">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Depend, 50>,
    VersionedClause<OMPC_NoWait, 51>,
````
- **L1345 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L1345 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L1346 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L1346 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L1347 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_InReduction>,`.
  **L1347 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_InReduction>,`。
- **L1348 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L1348 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L1349 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Mergeable>,`.
  **L1349 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Mergeable>,`。
- **L1350 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoGroup>,`.
  **L1350 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoGroup>,`。
- **L1351 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L1351 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L1352 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L1352 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L1353 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L1353 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L1354 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Untied>,`.
  **L1354 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Untied>,`。
- **L1355 EN**: Executes a standalone statement or declaration: `];`.
  **L1355 CN**: 执行一条独立语句或声明：`];`。
- **L1356 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1356 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1357 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L1357 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L1358 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L1358 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L1359 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Final>,`.
  **L1359 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Final>,`。
- **L1360 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L1360 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L1361 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Priority>,`.
  **L1361 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Priority>,`。
- **L1362 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Threadset, 60>,`.
  **L1362 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Threadset, 60>,`。
- **L1363 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Replayable, 60>,`.
  **L1363 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Replayable, 60>,`。
- **L1364 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Transparent, 60>,`.
  **L1364 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Transparent, 60>,`。
- **L1365 EN**: Executes a standalone statement or declaration: `];`.
  **L1365 CN**: 执行一条独立语句或声明：`];`。
- **L1366 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1366 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1367 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_GrainSize>,`.
  **L1367 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_GrainSize>,`。
- **L1368 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumTasks>,`.
  **L1368 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumTasks>,`。
- **L1369 EN**: Executes a standalone statement or declaration: `];`.
  **L1369 CN**: 执行一条独立语句或声明：`];`。
- **L1370 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1370 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1371 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1371 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1372 EN**: Closes the current lexical scope or compound statement.
  **L1372 CN**: 结束当前词法作用域或复合语句块。
- **L1373 EN**: Declares TableGen def `OMP_TaskWait`.
  **L1373 CN**: 声明 TableGen def `OMP_TaskWait`。
- **L1374 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1374 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1375 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Depend, 50>,`.
  **L1375 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Depend, 50>,`。
- **L1376 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoWait, 51>,`.
  **L1376 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoWait, 51>,`。

### Lines 1377-1408

````tablegen
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_Replayable, 60>,
  ];
  let association = AS_None;
  let category = CA_Executable;
}
def OMP_TaskYield : Directive<[Spelling<"taskyield">]> {
  let association = AS_None;
  let category = CA_Executable;
}
def OMP_Teams : Directive<[Spelling<"teams">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_DynGroupprivate, 61>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_OMPX_Attribute>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_Shared>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_If, 52>,
    VersionedClause<OMPC_NumTeams>,
    VersionedClause<OMPC_ThreadLimit>,
  ];
  let association = AS_Block;
  let category = CA_Executable;
}
def OMP_ThreadPrivate : Directive<[Spelling<"threadprivate">]> {
  let association = AS_None;
````
- **L1377 EN**: Executes a standalone statement or declaration: `];`.
  **L1377 CN**: 执行一条独立语句或声明：`];`。
- **L1378 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1378 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1379 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Replayable, 60>,`.
  **L1379 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Replayable, 60>,`。
- **L1380 EN**: Executes a standalone statement or declaration: `];`.
  **L1380 CN**: 执行一条独立语句或声明：`];`。
- **L1381 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1381 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1382 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1382 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1383 EN**: Closes the current lexical scope or compound statement.
  **L1383 CN**: 结束当前词法作用域或复合语句块。
- **L1384 EN**: Declares TableGen def `OMP_TaskYield`.
  **L1384 CN**: 声明 TableGen def `OMP_TaskYield`。
- **L1385 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1385 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1386 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1386 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1387 EN**: Closes the current lexical scope or compound statement.
  **L1387 CN**: 结束当前词法作用域或复合语句块。
- **L1388 EN**: Declares TableGen def `OMP_Teams`.
  **L1388 CN**: 声明 TableGen def `OMP_Teams`。
- **L1389 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1389 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1390 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L1390 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L1391 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DynGroupprivate, 61>,`.
  **L1391 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DynGroupprivate, 61>,`。
- **L1392 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L1392 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L1393 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_Attribute>,`.
  **L1393 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_Attribute>,`。
- **L1394 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L1394 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L1395 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L1395 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L1396 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L1396 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L1397 EN**: Executes a standalone statement or declaration: `];`.
  **L1397 CN**: 执行一条独立语句或声明：`];`。
- **L1398 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1398 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1399 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L1399 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L1400 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If, 52>,`.
  **L1400 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If, 52>,`。
- **L1401 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumTeams>,`.
  **L1401 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumTeams>,`。
- **L1402 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ThreadLimit>,`.
  **L1402 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ThreadLimit>,`。
- **L1403 EN**: Executes a standalone statement or declaration: `];`.
  **L1403 CN**: 执行一条独立语句或声明：`];`。
- **L1404 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1404 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1405 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1405 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1406 EN**: Closes the current lexical scope or compound statement.
  **L1406 CN**: 结束当前词法作用域或复合语句块。
- **L1407 EN**: Declares TableGen def `OMP_ThreadPrivate`.
  **L1407 CN**: 声明 TableGen def `OMP_ThreadPrivate`。
- **L1408 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1408 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 1409-1440

````tablegen
  let category = CA_Declarative;
}
def OMP_Tile : Directive<[Spelling<"tile">]> {
  let allowedOnceClauses = [
    VersionedClause<OMPC_Sizes, 51>,
  ];
  let requiredClauses = [
    VersionedClause<OMPC_Sizes, 51>,
  ];
  let association = AS_LoopNest;
  let category = CA_Executable;
}
def OMP_Stripe : Directive<[Spelling<"stripe">]> {
  let allowedOnceClauses = [
    VersionedClause<OMPC_Sizes, 60>,
  ];
  let association = AS_LoopNest;
  let category = CA_Executable;
}
def OMP_Split : Directive<[Spelling<"split">]> {
  let allowedOnceClauses = [
    VersionedClause<OMPC_Counts, 60>,
  ];
  let requiredClauses = [
    VersionedClause<OMPC_Counts, 60>,
  ];
  let association = AS_LoopNest;
  let category = CA_Executable;
}
def OMP_Unknown : Directive<[Spelling<"unknown">]> {
  let isDefault = true;
  let association = AS_None;
````
- **L1409 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1409 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1410 EN**: Closes the current lexical scope or compound statement.
  **L1410 CN**: 结束当前词法作用域或复合语句块。
- **L1411 EN**: Declares TableGen def `OMP_Tile`.
  **L1411 CN**: 声明 TableGen def `OMP_Tile`。
- **L1412 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1412 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1413 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Sizes, 51>,`.
  **L1413 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Sizes, 51>,`。
- **L1414 EN**: Executes a standalone statement or declaration: `];`.
  **L1414 CN**: 执行一条独立语句或声明：`];`。
- **L1415 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1415 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1416 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Sizes, 51>,`.
  **L1416 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Sizes, 51>,`。
- **L1417 EN**: Executes a standalone statement or declaration: `];`.
  **L1417 CN**: 执行一条独立语句或声明：`];`。
- **L1418 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1418 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1419 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1419 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1420 EN**: Closes the current lexical scope or compound statement.
  **L1420 CN**: 结束当前词法作用域或复合语句块。
- **L1421 EN**: Declares TableGen def `OMP_Stripe`.
  **L1421 CN**: 声明 TableGen def `OMP_Stripe`。
- **L1422 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1422 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1423 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Sizes, 60>,`.
  **L1423 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Sizes, 60>,`。
- **L1424 EN**: Executes a standalone statement or declaration: `];`.
  **L1424 CN**: 执行一条独立语句或声明：`];`。
- **L1425 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1425 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1426 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1426 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1427 EN**: Closes the current lexical scope or compound statement.
  **L1427 CN**: 结束当前词法作用域或复合语句块。
- **L1428 EN**: Declares TableGen def `OMP_Split`.
  **L1428 CN**: 声明 TableGen def `OMP_Split`。
- **L1429 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1429 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1430 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Counts, 60>,`.
  **L1430 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Counts, 60>,`。
- **L1431 EN**: Executes a standalone statement or declaration: `];`.
  **L1431 CN**: 执行一条独立语句或声明：`];`。
- **L1432 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1432 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1433 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Counts, 60>,`.
  **L1433 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Counts, 60>,`。
- **L1434 EN**: Executes a standalone statement or declaration: `];`.
  **L1434 CN**: 执行一条独立语句或声明：`];`。
- **L1435 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1435 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1436 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1436 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1437 EN**: Closes the current lexical scope or compound statement.
  **L1437 CN**: 结束当前词法作用域或复合语句块。
- **L1438 EN**: Declares TableGen def `OMP_Unknown`.
  **L1438 CN**: 声明 TableGen def `OMP_Unknown`。
- **L1439 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1439 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1440 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1440 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 1441-1472

````tablegen
  let category = CA_Utility;
}
def OMP_Unroll : Directive<[Spelling<"unroll">]> {
  let allowedOnceClauses = [
    VersionedClause<OMPC_Full, 51>,
    VersionedClause<OMPC_Partial, 51>,
  ];
  let association = AS_LoopNest;
  let category = CA_Executable;
}
def OMP_Workshare : Directive<[Spelling<"workshare">]> {
  let allowedOnceClauses = [
    VersionedClause<OMPC_NoWait>,
  ];
  let association = AS_Block;
  let category = CA_Executable;
  let languages = [L_Fortran];
}
def OMP_EndWorkshare : Directive<[Spelling<"end workshare">]> {
  let allowedOnceClauses = [
    VersionedClause<OMPC_NoWait>,
  ];
  let leafConstructs = OMP_Workshare.leafConstructs;
  let association = OMP_Workshare.association;
  let category = OMP_Workshare.category;
  let languages = [L_Fortran];
}
def OMP_Workdistribute : Directive<[Spelling<"workdistribute">]> {
  let association = AS_Block;
  let category = CA_Executable;
  let languages = [L_Fortran];
}
````
- **L1441 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1441 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1442 EN**: Closes the current lexical scope or compound statement.
  **L1442 CN**: 结束当前词法作用域或复合语句块。
- **L1443 EN**: Declares TableGen def `OMP_Unroll`.
  **L1443 CN**: 声明 TableGen def `OMP_Unroll`。
- **L1444 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1444 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1445 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Full, 51>,`.
  **L1445 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Full, 51>,`。
- **L1446 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Partial, 51>,`.
  **L1446 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Partial, 51>,`。
- **L1447 EN**: Executes a standalone statement or declaration: `];`.
  **L1447 CN**: 执行一条独立语句或声明：`];`。
- **L1448 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1448 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1449 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1449 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1450 EN**: Closes the current lexical scope or compound statement.
  **L1450 CN**: 结束当前词法作用域或复合语句块。
- **L1451 EN**: Declares TableGen def `OMP_Workshare`.
  **L1451 CN**: 声明 TableGen def `OMP_Workshare`。
- **L1452 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1452 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1453 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoWait>,`.
  **L1453 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoWait>,`。
- **L1454 EN**: Executes a standalone statement or declaration: `];`.
  **L1454 CN**: 执行一条独立语句或声明：`];`。
- **L1455 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1455 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1456 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1456 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1457 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1457 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1458 EN**: Closes the current lexical scope or compound statement.
  **L1458 CN**: 结束当前词法作用域或复合语句块。
- **L1459 EN**: Declares TableGen def `OMP_EndWorkshare`.
  **L1459 CN**: 声明 TableGen def `OMP_EndWorkshare`。
- **L1460 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1460 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1461 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoWait>,`.
  **L1461 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoWait>,`。
- **L1462 EN**: Executes a standalone statement or declaration: `];`.
  **L1462 CN**: 执行一条独立语句或声明：`];`。
- **L1463 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1463 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1464 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1464 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1465 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1465 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1466 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1466 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1467 EN**: Closes the current lexical scope or compound statement.
  **L1467 CN**: 结束当前词法作用域或复合语句块。
- **L1468 EN**: Declares TableGen def `OMP_Workdistribute`.
  **L1468 CN**: 声明 TableGen def `OMP_Workdistribute`。
- **L1469 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1469 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1470 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1470 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1471 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1471 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1472 EN**: Closes the current lexical scope or compound statement.
  **L1472 CN**: 结束当前词法作用域或复合语句块。

### Lines 1473-1504

````tablegen
def OMP_EndWorkdistribute : Directive<[Spelling<"end workdistribute">]> {
  let leafConstructs = OMP_Workdistribute.leafConstructs;
  let association = OMP_Workdistribute.association;
  let category = OMP_Workdistribute.category;
  let languages = [L_Fortran];
}

//===----------------------------------------------------------------------===//
// Definitions of OpenMP compound directives
// Sorted alphabetically wrt directive spelling, except "end xyz" immediately
// follows "xyz".
//===----------------------------------------------------------------------===//

def OMP_DistributeParallelDo : Directive<[Spelling<"distribute parallel do">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Copyin>,
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Linear>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_Shared>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_DistSchedule>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_Message, 60>,
    VersionedClause<OMPC_NumThreads>,
    VersionedClause<OMPC_Order, 50>,
````
- **L1473 EN**: Declares TableGen def `OMP_EndWorkdistribute`.
  **L1473 CN**: 声明 TableGen def `OMP_EndWorkdistribute`。
- **L1474 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1474 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1475 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1475 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1476 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1476 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1477 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1477 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1478 EN**: Closes the current lexical scope or compound statement.
  **L1478 CN**: 结束当前词法作用域或复合语句块。
- **L1479 EN**: Blank line separating nearby declarations or logic blocks.
  **L1479 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1480 EN**: Banner comment marking a file or section boundary.
  **L1480 CN**: 横幅注释，用于标记文件或章节边界。
- **L1481 EN**: Comment explains nearby logic, invariants, or intent: `Definitions of OpenMP compound directives`.
  **L1481 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Definitions of OpenMP compound directives`。
- **L1482 EN**: Comment explains nearby logic, invariants, or intent: `Sorted alphabetically wrt directive spelling, except "end xyz" immediately`.
  **L1482 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Sorted alphabetically wrt directive spelling, except "end xyz" immediately`。
- **L1483 EN**: Comment explains nearby logic, invariants, or intent: `follows "xyz".`.
  **L1483 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`follows "xyz".`。
- **L1484 EN**: Banner comment marking a file or section boundary.
  **L1484 CN**: 横幅注释，用于标记文件或章节边界。
- **L1485 EN**: Blank line separating nearby declarations or logic blocks.
  **L1485 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1486 EN**: Declares TableGen def `OMP_DistributeParallelDo`.
  **L1486 CN**: 声明 TableGen def `OMP_DistributeParallelDo`。
- **L1487 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1487 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1488 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L1488 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L1489 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Copyin>,`.
  **L1489 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Copyin>,`。
- **L1490 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L1490 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L1491 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L1491 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L1492 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L1492 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L1493 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Linear>,`.
  **L1493 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Linear>,`。
- **L1494 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L1494 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L1495 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L1495 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L1496 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L1496 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L1497 EN**: Executes a standalone statement or declaration: `];`.
  **L1497 CN**: 执行一条独立语句或声明：`];`。
- **L1498 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1498 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1499 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L1499 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L1500 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DistSchedule>,`.
  **L1500 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DistSchedule>,`。
- **L1501 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L1501 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L1502 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Message, 60>,`.
  **L1502 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Message, 60>,`。
- **L1503 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumThreads>,`.
  **L1503 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumThreads>,`。
- **L1504 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Order, 50>,`.
  **L1504 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Order, 50>,`。

### Lines 1505-1536

````tablegen
    VersionedClause<OMPC_ProcBind>,
    VersionedClause<OMPC_Schedule>,
    VersionedClause<OMPC_Severity, 60>,
  ];
  let leafConstructs = [OMP_Distribute, OMP_Parallel, OMP_Do];
  let category = CA_Executable;
  let languages = [L_Fortran];
}
def OMP_DistributeParallelDoSimd
    : Directive<[Spelling<"distribute parallel do simd">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Aligned>,
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_Copyin>,
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_DistSchedule>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Linear>,
    VersionedClause<OMPC_Message, 60>,
    VersionedClause<OMPC_NonTemporal>,
    VersionedClause<OMPC_NumThreads>,
    VersionedClause<OMPC_Order, 50>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_ProcBind>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_SafeLen>,
    VersionedClause<OMPC_Schedule>,
    VersionedClause<OMPC_Severity, 60>,
    VersionedClause<OMPC_Shared>,
````
- **L1505 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ProcBind>,`.
  **L1505 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ProcBind>,`。
- **L1506 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Schedule>,`.
  **L1506 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Schedule>,`。
- **L1507 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Severity, 60>,`.
  **L1507 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Severity, 60>,`。
- **L1508 EN**: Executes a standalone statement or declaration: `];`.
  **L1508 CN**: 执行一条独立语句或声明：`];`。
- **L1509 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1509 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1510 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1510 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1511 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1511 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1512 EN**: Closes the current lexical scope or compound statement.
  **L1512 CN**: 结束当前词法作用域或复合语句块。
- **L1513 EN**: Declares TableGen def `OMP_DistributeParallelDoSimd`.
  **L1513 CN**: 声明 TableGen def `OMP_DistributeParallelDoSimd`。
- **L1514 EN**: Continues the surrounding expression or declaration: `: Directive<[Spelling<"distribute parallel do simd">]> {`.
  **L1514 CN**: 继续构造周围的表达式或声明：`: Directive<[Spelling<"distribute parallel do simd">]> {`。
- **L1515 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1515 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1516 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Aligned>,`.
  **L1516 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Aligned>,`。
- **L1517 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L1517 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L1518 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L1518 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L1519 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Copyin>,`.
  **L1519 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Copyin>,`。
- **L1520 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L1520 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L1521 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DistSchedule>,`.
  **L1521 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DistSchedule>,`。
- **L1522 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L1522 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L1523 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L1523 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L1524 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L1524 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L1525 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Linear>,`.
  **L1525 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Linear>,`。
- **L1526 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Message, 60>,`.
  **L1526 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Message, 60>,`。
- **L1527 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NonTemporal>,`.
  **L1527 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NonTemporal>,`。
- **L1528 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumThreads>,`.
  **L1528 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumThreads>,`。
- **L1529 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Order, 50>,`.
  **L1529 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Order, 50>,`。
- **L1530 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L1530 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L1531 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ProcBind>,`.
  **L1531 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ProcBind>,`。
- **L1532 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L1532 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L1533 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SafeLen>,`.
  **L1533 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SafeLen>,`。
- **L1534 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Schedule>,`.
  **L1534 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Schedule>,`。
- **L1535 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Severity, 60>,`.
  **L1535 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Severity, 60>,`。
- **L1536 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L1536 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。

### Lines 1537-1568

````tablegen
    VersionedClause<OMPC_SimdLen>,
  ];
  let leafConstructs = [OMP_Distribute, OMP_Parallel, OMP_Do, OMP_Simd];
  let category = CA_Executable;
  let languages = [L_Fortran];
}
def OMP_DistributeParallelFor
    : Directive<[Spelling<"distribute parallel for">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_Copyin>,
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_DistSchedule>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Message, 60>,
    VersionedClause<OMPC_NumThreads>,
    VersionedClause<OMPC_OMPX_Attribute>,
    VersionedClause<OMPC_Order, 50>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_ProcBind>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_Schedule>,
    VersionedClause<OMPC_Severity, 60>,
    VersionedClause<OMPC_Shared>,
  ];
  let leafConstructs = [OMP_Distribute, OMP_Parallel, OMP_For];
  let category = CA_Executable;
  let languages = [L_C];
}
````
- **L1537 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SimdLen>,`.
  **L1537 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SimdLen>,`。
- **L1538 EN**: Executes a standalone statement or declaration: `];`.
  **L1538 CN**: 执行一条独立语句或声明：`];`。
- **L1539 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1539 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1540 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1540 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1541 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1541 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1542 EN**: Closes the current lexical scope or compound statement.
  **L1542 CN**: 结束当前词法作用域或复合语句块。
- **L1543 EN**: Declares TableGen def `OMP_DistributeParallelFor`.
  **L1543 CN**: 声明 TableGen def `OMP_DistributeParallelFor`。
- **L1544 EN**: Continues the surrounding expression or declaration: `: Directive<[Spelling<"distribute parallel for">]> {`.
  **L1544 CN**: 继续构造周围的表达式或声明：`: Directive<[Spelling<"distribute parallel for">]> {`。
- **L1545 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1545 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1546 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L1546 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L1547 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L1547 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L1548 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Copyin>,`.
  **L1548 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Copyin>,`。
- **L1549 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L1549 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L1550 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DistSchedule>,`.
  **L1550 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DistSchedule>,`。
- **L1551 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L1551 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L1552 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L1552 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L1553 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L1553 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L1554 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Message, 60>,`.
  **L1554 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Message, 60>,`。
- **L1555 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumThreads>,`.
  **L1555 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumThreads>,`。
- **L1556 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_Attribute>,`.
  **L1556 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_Attribute>,`。
- **L1557 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Order, 50>,`.
  **L1557 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Order, 50>,`。
- **L1558 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L1558 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L1559 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ProcBind>,`.
  **L1559 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ProcBind>,`。
- **L1560 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L1560 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L1561 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Schedule>,`.
  **L1561 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Schedule>,`。
- **L1562 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Severity, 60>,`.
  **L1562 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Severity, 60>,`。
- **L1563 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L1563 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L1564 EN**: Executes a standalone statement or declaration: `];`.
  **L1564 CN**: 执行一条独立语句或声明：`];`。
- **L1565 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1565 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1566 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1566 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1567 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1567 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1568 EN**: Closes the current lexical scope or compound statement.
  **L1568 CN**: 结束当前词法作用域或复合语句块。

### Lines 1569-1600

````tablegen
def OMP_DistributeParallelForSimd
    : Directive<[Spelling<"distribute parallel for simd">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Aligned>,
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_Copyin>,
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_DistSchedule>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Linear>,
    VersionedClause<OMPC_Message, 60>,
    VersionedClause<OMPC_NonTemporal, 50>,
    VersionedClause<OMPC_NumThreads>,
    VersionedClause<OMPC_OMPX_Attribute>,
    VersionedClause<OMPC_Order, 50>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_ProcBind>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_SafeLen>,
    VersionedClause<OMPC_Schedule>,
    VersionedClause<OMPC_Severity, 60>,
    VersionedClause<OMPC_Shared>,
    VersionedClause<OMPC_SimdLen>,
  ];
  let leafConstructs = [OMP_Distribute, OMP_Parallel, OMP_For, OMP_Simd];
  let category = CA_Executable;
  let languages = [L_C];
}
def OMP_DistributeSimd : Directive<[Spelling<"distribute simd">]> {
````
- **L1569 EN**: Declares TableGen def `OMP_DistributeParallelForSimd`.
  **L1569 CN**: 声明 TableGen def `OMP_DistributeParallelForSimd`。
- **L1570 EN**: Continues the surrounding expression or declaration: `: Directive<[Spelling<"distribute parallel for simd">]> {`.
  **L1570 CN**: 继续构造周围的表达式或声明：`: Directive<[Spelling<"distribute parallel for simd">]> {`。
- **L1571 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1571 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1572 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Aligned>,`.
  **L1572 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Aligned>,`。
- **L1573 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L1573 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L1574 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L1574 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L1575 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Copyin>,`.
  **L1575 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Copyin>,`。
- **L1576 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L1576 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L1577 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DistSchedule>,`.
  **L1577 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DistSchedule>,`。
- **L1578 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L1578 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L1579 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L1579 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L1580 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L1580 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L1581 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Linear>,`.
  **L1581 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Linear>,`。
- **L1582 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Message, 60>,`.
  **L1582 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Message, 60>,`。
- **L1583 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NonTemporal, 50>,`.
  **L1583 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NonTemporal, 50>,`。
- **L1584 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumThreads>,`.
  **L1584 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumThreads>,`。
- **L1585 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_Attribute>,`.
  **L1585 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_Attribute>,`。
- **L1586 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Order, 50>,`.
  **L1586 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Order, 50>,`。
- **L1587 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L1587 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L1588 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ProcBind>,`.
  **L1588 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ProcBind>,`。
- **L1589 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L1589 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L1590 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SafeLen>,`.
  **L1590 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SafeLen>,`。
- **L1591 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Schedule>,`.
  **L1591 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Schedule>,`。
- **L1592 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Severity, 60>,`.
  **L1592 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Severity, 60>,`。
- **L1593 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L1593 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L1594 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SimdLen>,`.
  **L1594 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SimdLen>,`。
- **L1595 EN**: Executes a standalone statement or declaration: `];`.
  **L1595 CN**: 执行一条独立语句或声明：`];`。
- **L1596 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1596 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1597 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1597 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1598 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1598 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1599 EN**: Closes the current lexical scope or compound statement.
  **L1599 CN**: 结束当前词法作用域或复合语句块。
- **L1600 EN**: Declares TableGen def `OMP_DistributeSimd`.
  **L1600 CN**: 声明 TableGen def `OMP_DistributeSimd`。

### Lines 1601-1632

````tablegen
  let allowedClauses = [
    VersionedClause<OMPC_Aligned>,
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Copyin>,
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Linear>,
    VersionedClause<OMPC_NonTemporal, 50>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_Reduction>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_DistSchedule>,
    VersionedClause<OMPC_If, 50>,
    VersionedClause<OMPC_Message, 60>,
    VersionedClause<OMPC_NumThreads>,
    VersionedClause<OMPC_Order, 50>,
    VersionedClause<OMPC_ProcBind>,
    VersionedClause<OMPC_SafeLen>,
    VersionedClause<OMPC_Schedule>,
    VersionedClause<OMPC_Severity, 60>,
    VersionedClause<OMPC_SimdLen>,
  ];
  let leafConstructs = [OMP_Distribute, OMP_Simd];
  let category = CA_Executable;
}
def OMP_DoSimd : Directive<[Spelling<"do simd">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Aligned>,
    VersionedClause<OMPC_FirstPrivate>,
````
- **L1601 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1601 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1602 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Aligned>,`.
  **L1602 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Aligned>,`。
- **L1603 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L1603 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L1604 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Copyin>,`.
  **L1604 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Copyin>,`。
- **L1605 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L1605 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L1606 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L1606 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L1607 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L1607 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L1608 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Linear>,`.
  **L1608 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Linear>,`。
- **L1609 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NonTemporal, 50>,`.
  **L1609 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NonTemporal, 50>,`。
- **L1610 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L1610 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L1611 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L1611 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L1612 EN**: Executes a standalone statement or declaration: `];`.
  **L1612 CN**: 执行一条独立语句或声明：`];`。
- **L1613 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1613 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1614 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L1614 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L1615 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DistSchedule>,`.
  **L1615 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DistSchedule>,`。
- **L1616 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If, 50>,`.
  **L1616 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If, 50>,`。
- **L1617 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Message, 60>,`.
  **L1617 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Message, 60>,`。
- **L1618 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumThreads>,`.
  **L1618 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumThreads>,`。
- **L1619 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Order, 50>,`.
  **L1619 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Order, 50>,`。
- **L1620 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ProcBind>,`.
  **L1620 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ProcBind>,`。
- **L1621 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SafeLen>,`.
  **L1621 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SafeLen>,`。
- **L1622 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Schedule>,`.
  **L1622 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Schedule>,`。
- **L1623 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Severity, 60>,`.
  **L1623 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Severity, 60>,`。
- **L1624 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SimdLen>,`.
  **L1624 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SimdLen>,`。
- **L1625 EN**: Executes a standalone statement or declaration: `];`.
  **L1625 CN**: 执行一条独立语句或声明：`];`。
- **L1626 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1626 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1627 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1627 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1628 EN**: Closes the current lexical scope or compound statement.
  **L1628 CN**: 结束当前词法作用域或复合语句块。
- **L1629 EN**: Declares TableGen def `OMP_DoSimd`.
  **L1629 CN**: 声明 TableGen def `OMP_DoSimd`。
- **L1630 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1630 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1631 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Aligned>,`.
  **L1631 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Aligned>,`。
- **L1632 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L1632 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。

### Lines 1633-1664

````tablegen
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Linear>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_Reduction>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_If, 50>,
    VersionedClause<OMPC_NoWait>,
    VersionedClause<OMPC_Order, 50>,
    VersionedClause<OMPC_Ordered>,
    VersionedClause<OMPC_SafeLen>,
    VersionedClause<OMPC_Schedule>,
    VersionedClause<OMPC_SimdLen>,
  ];
  let leafConstructs = [OMP_Do, OMP_Simd];
  let category = CA_Executable;
  let languages = [L_Fortran];
}
def OMP_EndDoSimd : Directive<[Spelling<"end do simd">]> {
  let allowedOnceClauses = [
    VersionedClause<OMPC_NoWait>,
  ];
  let leafConstructs = OMP_DoSimd.leafConstructs;
  let association = OMP_DoSimd.association;
  let category = OMP_DoSimd.category;
  let languages = [L_Fortran];
}
def OMP_ForSimd : Directive<[Spelling<"for simd">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Aligned>,
    VersionedClause<OMPC_Allocate>,
````
- **L1633 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L1633 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L1634 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Linear>,`.
  **L1634 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Linear>,`。
- **L1635 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L1635 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L1636 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L1636 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L1637 EN**: Executes a standalone statement or declaration: `];`.
  **L1637 CN**: 执行一条独立语句或声明：`];`。
- **L1638 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1638 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1639 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L1639 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L1640 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If, 50>,`.
  **L1640 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If, 50>,`。
- **L1641 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoWait>,`.
  **L1641 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoWait>,`。
- **L1642 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Order, 50>,`.
  **L1642 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Order, 50>,`。
- **L1643 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Ordered>,`.
  **L1643 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Ordered>,`。
- **L1644 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SafeLen>,`.
  **L1644 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SafeLen>,`。
- **L1645 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Schedule>,`.
  **L1645 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Schedule>,`。
- **L1646 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SimdLen>,`.
  **L1646 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SimdLen>,`。
- **L1647 EN**: Executes a standalone statement or declaration: `];`.
  **L1647 CN**: 执行一条独立语句或声明：`];`。
- **L1648 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1648 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1649 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1649 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1650 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1650 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1651 EN**: Closes the current lexical scope or compound statement.
  **L1651 CN**: 结束当前词法作用域或复合语句块。
- **L1652 EN**: Declares TableGen def `OMP_EndDoSimd`.
  **L1652 CN**: 声明 TableGen def `OMP_EndDoSimd`。
- **L1653 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1653 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1654 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoWait>,`.
  **L1654 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoWait>,`。
- **L1655 EN**: Executes a standalone statement or declaration: `];`.
  **L1655 CN**: 执行一条独立语句或声明：`];`。
- **L1656 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1656 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1657 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1657 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1658 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1658 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1659 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1659 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1660 EN**: Closes the current lexical scope or compound statement.
  **L1660 CN**: 结束当前词法作用域或复合语句块。
- **L1661 EN**: Declares TableGen def `OMP_ForSimd`.
  **L1661 CN**: 声明 TableGen def `OMP_ForSimd`。
- **L1662 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1662 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1663 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Aligned>,`.
  **L1663 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Aligned>,`。
- **L1664 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L1664 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。

### Lines 1665-1696

````tablegen
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_If, 50>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Linear>,
    VersionedClause<OMPC_NonTemporal, 50>,
    VersionedClause<OMPC_NoWait>,
    VersionedClause<OMPC_Order, 50>,
    VersionedClause<OMPC_Ordered>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_SafeLen>,
    VersionedClause<OMPC_Schedule>,
    VersionedClause<OMPC_SimdLen>,
  ];
  let leafConstructs = [OMP_For, OMP_Simd];
  let category = CA_Executable;
}
def OMP_target_loop : Directive<[Spelling<"target loop">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Depend>,
    VersionedClause<OMPC_DynGroupprivate, 61>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_IsDevicePtr>,
    VersionedClause<OMPC_HasDeviceAddr, 51>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Map>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_UsesAllocators, 50>,
    VersionedClause<OMPC_OMPX_Attribute>,
````
- **L1665 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L1665 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L1666 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L1666 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L1667 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If, 50>,`.
  **L1667 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If, 50>,`。
- **L1668 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L1668 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L1669 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Linear>,`.
  **L1669 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Linear>,`。
- **L1670 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NonTemporal, 50>,`.
  **L1670 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NonTemporal, 50>,`。
- **L1671 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoWait>,`.
  **L1671 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoWait>,`。
- **L1672 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Order, 50>,`.
  **L1672 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Order, 50>,`。
- **L1673 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Ordered>,`.
  **L1673 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Ordered>,`。
- **L1674 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L1674 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L1675 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L1675 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L1676 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SafeLen>,`.
  **L1676 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SafeLen>,`。
- **L1677 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Schedule>,`.
  **L1677 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Schedule>,`。
- **L1678 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SimdLen>,`.
  **L1678 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SimdLen>,`。
- **L1679 EN**: Executes a standalone statement or declaration: `];`.
  **L1679 CN**: 执行一条独立语句或声明：`];`。
- **L1680 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1680 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1681 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1681 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1682 EN**: Closes the current lexical scope or compound statement.
  **L1682 CN**: 结束当前词法作用域或复合语句块。
- **L1683 EN**: Declares TableGen def `OMP_target_loop`.
  **L1683 CN**: 声明 TableGen def `OMP_target_loop`。
- **L1684 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1684 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1685 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L1685 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L1686 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Depend>,`.
  **L1686 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Depend>,`。
- **L1687 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DynGroupprivate, 61>,`.
  **L1687 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DynGroupprivate, 61>,`。
- **L1688 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L1688 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L1689 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_IsDevicePtr>,`.
  **L1689 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_IsDevicePtr>,`。
- **L1690 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_HasDeviceAddr, 51>,`.
  **L1690 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_HasDeviceAddr, 51>,`。
- **L1691 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L1691 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L1692 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Map>,`.
  **L1692 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Map>,`。
- **L1693 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L1693 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L1694 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L1694 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L1695 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_UsesAllocators, 50>,`.
  **L1695 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_UsesAllocators, 50>,`。
- **L1696 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_Attribute>,`.
  **L1696 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_Attribute>,`。

### Lines 1697-1728

````tablegen
    VersionedClause<OMPC_InReduction, 50>,

  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_Bind, 50>,
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_Order>,
    VersionedClause<OMPC_ThreadLimit>,
    VersionedClause<OMPC_OMPX_DynCGroupMem>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_Device>,
    VersionedClause<OMPC_DefaultMap>,
    VersionedClause<OMPC_NoWait>,
  ];
  let leafConstructs = [OMP_Target, OMP_loop];
  let category = CA_Executable;
}
def OMP_MaskedTaskloop : Directive<[Spelling<"masked taskloop">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_Filter>,
    VersionedClause<OMPC_Final>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_GrainSize>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_InReduction>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Mergeable>,
    VersionedClause<OMPC_NoGroup>,
    VersionedClause<OMPC_NumTasks>,
````
- **L1697 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_InReduction, 50>,`.
  **L1697 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_InReduction, 50>,`。
- **L1698 EN**: Blank line separating nearby declarations or logic blocks.
  **L1698 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1699 EN**: Executes a standalone statement or declaration: `];`.
  **L1699 CN**: 执行一条独立语句或声明：`];`。
- **L1700 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1700 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1701 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Bind, 50>,`.
  **L1701 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Bind, 50>,`。
- **L1702 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L1702 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L1703 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Order>,`.
  **L1703 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Order>,`。
- **L1704 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ThreadLimit>,`.
  **L1704 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ThreadLimit>,`。
- **L1705 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_DynCGroupMem>,`.
  **L1705 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_DynCGroupMem>,`。
- **L1706 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L1706 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L1707 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Device>,`.
  **L1707 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Device>,`。
- **L1708 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DefaultMap>,`.
  **L1708 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DefaultMap>,`。
- **L1709 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoWait>,`.
  **L1709 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoWait>,`。
- **L1710 EN**: Executes a standalone statement or declaration: `];`.
  **L1710 CN**: 执行一条独立语句或声明：`];`。
- **L1711 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1711 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1712 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1712 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1713 EN**: Closes the current lexical scope or compound statement.
  **L1713 CN**: 结束当前词法作用域或复合语句块。
- **L1714 EN**: Declares TableGen def `OMP_MaskedTaskloop`.
  **L1714 CN**: 声明 TableGen def `OMP_MaskedTaskloop`。
- **L1715 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1715 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1716 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L1716 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L1717 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L1717 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L1718 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L1718 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L1719 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Filter>,`.
  **L1719 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Filter>,`。
- **L1720 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Final>,`.
  **L1720 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Final>,`。
- **L1721 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L1721 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L1722 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_GrainSize>,`.
  **L1722 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_GrainSize>,`。
- **L1723 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L1723 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L1724 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_InReduction>,`.
  **L1724 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_InReduction>,`。
- **L1725 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L1725 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L1726 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Mergeable>,`.
  **L1726 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Mergeable>,`。
- **L1727 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoGroup>,`.
  **L1727 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoGroup>,`。
- **L1728 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumTasks>,`.
  **L1728 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumTasks>,`。

### Lines 1729-1760

````tablegen
    VersionedClause<OMPC_Priority>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_Shared>,
    VersionedClause<OMPC_Untied>,
  ];
  let leafConstructs = [OMP_masked, OMP_TaskLoop];
  let category = CA_Executable;
}
def OMP_MaskedTaskloopSimd : Directive<[Spelling<"masked taskloop simd">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Aligned>,
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_Filter>,
    VersionedClause<OMPC_Final>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_GrainSize>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_InReduction>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Linear>,
    VersionedClause<OMPC_Mergeable>,
    VersionedClause<OMPC_NoGroup>,
    VersionedClause<OMPC_NonTemporal, 50>,
    VersionedClause<OMPC_NumTasks>,
    VersionedClause<OMPC_Order, 50>,
    VersionedClause<OMPC_Priority>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_SafeLen>,
````
- **L1729 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Priority>,`.
  **L1729 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Priority>,`。
- **L1730 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L1730 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L1731 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L1731 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L1732 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L1732 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L1733 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Untied>,`.
  **L1733 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Untied>,`。
- **L1734 EN**: Executes a standalone statement or declaration: `];`.
  **L1734 CN**: 执行一条独立语句或声明：`];`。
- **L1735 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1735 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1736 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1736 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1737 EN**: Closes the current lexical scope or compound statement.
  **L1737 CN**: 结束当前词法作用域或复合语句块。
- **L1738 EN**: Declares TableGen def `OMP_MaskedTaskloopSimd`.
  **L1738 CN**: 声明 TableGen def `OMP_MaskedTaskloopSimd`。
- **L1739 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1739 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1740 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Aligned>,`.
  **L1740 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Aligned>,`。
- **L1741 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L1741 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L1742 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L1742 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L1743 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L1743 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L1744 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Filter>,`.
  **L1744 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Filter>,`。
- **L1745 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Final>,`.
  **L1745 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Final>,`。
- **L1746 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L1746 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L1747 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_GrainSize>,`.
  **L1747 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_GrainSize>,`。
- **L1748 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L1748 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L1749 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_InReduction>,`.
  **L1749 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_InReduction>,`。
- **L1750 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L1750 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L1751 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Linear>,`.
  **L1751 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Linear>,`。
- **L1752 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Mergeable>,`.
  **L1752 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Mergeable>,`。
- **L1753 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoGroup>,`.
  **L1753 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoGroup>,`。
- **L1754 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NonTemporal, 50>,`.
  **L1754 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NonTemporal, 50>,`。
- **L1755 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumTasks>,`.
  **L1755 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumTasks>,`。
- **L1756 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Order, 50>,`.
  **L1756 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Order, 50>,`。
- **L1757 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Priority>,`.
  **L1757 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Priority>,`。
- **L1758 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L1758 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L1759 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L1759 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L1760 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SafeLen>,`.
  **L1760 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SafeLen>,`。

### Lines 1761-1792

````tablegen
    VersionedClause<OMPC_Shared>,
    VersionedClause<OMPC_SimdLen>,
    VersionedClause<OMPC_Untied>,
  ];
  let leafConstructs = [OMP_masked, OMP_TaskLoop, OMP_Simd];
  let category = CA_Executable;
}
def OMP_MasterTaskloop : Directive<[Spelling<"master taskloop">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_Final>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_GrainSize>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_InReduction>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Mergeable>,
    VersionedClause<OMPC_NoGroup>,
    VersionedClause<OMPC_NumTasks>,
    VersionedClause<OMPC_Priority>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_Shared>,
    VersionedClause<OMPC_Untied>,
  ];
  let leafConstructs = [OMP_Master, OMP_TaskLoop];
  let category = CA_Executable;
}
def OMP_MasterTaskloopSimd : Directive<[Spelling<"master taskloop simd">]> {
  let allowedClauses = [
````
- **L1761 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L1761 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L1762 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SimdLen>,`.
  **L1762 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SimdLen>,`。
- **L1763 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Untied>,`.
  **L1763 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Untied>,`。
- **L1764 EN**: Executes a standalone statement or declaration: `];`.
  **L1764 CN**: 执行一条独立语句或声明：`];`。
- **L1765 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1765 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1766 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1766 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1767 EN**: Closes the current lexical scope or compound statement.
  **L1767 CN**: 结束当前词法作用域或复合语句块。
- **L1768 EN**: Declares TableGen def `OMP_MasterTaskloop`.
  **L1768 CN**: 声明 TableGen def `OMP_MasterTaskloop`。
- **L1769 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1769 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1770 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L1770 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L1771 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L1771 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L1772 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L1772 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L1773 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Final>,`.
  **L1773 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Final>,`。
- **L1774 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L1774 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L1775 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_GrainSize>,`.
  **L1775 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_GrainSize>,`。
- **L1776 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L1776 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L1777 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_InReduction>,`.
  **L1777 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_InReduction>,`。
- **L1778 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L1778 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L1779 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Mergeable>,`.
  **L1779 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Mergeable>,`。
- **L1780 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoGroup>,`.
  **L1780 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoGroup>,`。
- **L1781 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumTasks>,`.
  **L1781 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumTasks>,`。
- **L1782 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Priority>,`.
  **L1782 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Priority>,`。
- **L1783 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L1783 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L1784 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L1784 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L1785 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L1785 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L1786 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Untied>,`.
  **L1786 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Untied>,`。
- **L1787 EN**: Executes a standalone statement or declaration: `];`.
  **L1787 CN**: 执行一条独立语句或声明：`];`。
- **L1788 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1788 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1789 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1789 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1790 EN**: Closes the current lexical scope or compound statement.
  **L1790 CN**: 结束当前词法作用域或复合语句块。
- **L1791 EN**: Declares TableGen def `OMP_MasterTaskloopSimd`.
  **L1791 CN**: 声明 TableGen def `OMP_MasterTaskloopSimd`。
- **L1792 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1792 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 1793-1824

````tablegen
    VersionedClause<OMPC_Aligned>,
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_Final>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_GrainSize>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_InReduction>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Linear>,
    VersionedClause<OMPC_Mergeable>,
    VersionedClause<OMPC_NoGroup>,
    VersionedClause<OMPC_NonTemporal, 50>,
    VersionedClause<OMPC_NumTasks>,
    VersionedClause<OMPC_Order, 50>,
    VersionedClause<OMPC_Priority>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_SafeLen>,
    VersionedClause<OMPC_Shared>,
    VersionedClause<OMPC_SimdLen>,
    VersionedClause<OMPC_Untied>,
  ];
  let leafConstructs = [OMP_Master, OMP_TaskLoop, OMP_Simd];
  let category = CA_Executable;
}
def OMP_ParallelDo : Directive<[Spelling<"parallel do">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Allocate, 50>,
    VersionedClause<OMPC_Copyin>,
    VersionedClause<OMPC_Default>,
````
- **L1793 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Aligned>,`.
  **L1793 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Aligned>,`。
- **L1794 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L1794 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L1795 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L1795 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L1796 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L1796 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L1797 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Final>,`.
  **L1797 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Final>,`。
- **L1798 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L1798 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L1799 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_GrainSize>,`.
  **L1799 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_GrainSize>,`。
- **L1800 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L1800 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L1801 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_InReduction>,`.
  **L1801 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_InReduction>,`。
- **L1802 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L1802 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L1803 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Linear>,`.
  **L1803 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Linear>,`。
- **L1804 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Mergeable>,`.
  **L1804 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Mergeable>,`。
- **L1805 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoGroup>,`.
  **L1805 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoGroup>,`。
- **L1806 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NonTemporal, 50>,`.
  **L1806 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NonTemporal, 50>,`。
- **L1807 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumTasks>,`.
  **L1807 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumTasks>,`。
- **L1808 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Order, 50>,`.
  **L1808 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Order, 50>,`。
- **L1809 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Priority>,`.
  **L1809 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Priority>,`。
- **L1810 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L1810 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L1811 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L1811 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L1812 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SafeLen>,`.
  **L1812 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SafeLen>,`。
- **L1813 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L1813 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L1814 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SimdLen>,`.
  **L1814 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SimdLen>,`。
- **L1815 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Untied>,`.
  **L1815 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Untied>,`。
- **L1816 EN**: Executes a standalone statement or declaration: `];`.
  **L1816 CN**: 执行一条独立语句或声明：`];`。
- **L1817 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1817 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1818 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1818 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1819 EN**: Closes the current lexical scope or compound statement.
  **L1819 CN**: 结束当前词法作用域或复合语句块。
- **L1820 EN**: Declares TableGen def `OMP_ParallelDo`.
  **L1820 CN**: 声明 TableGen def `OMP_ParallelDo`。
- **L1821 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1821 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1822 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate, 50>,`.
  **L1822 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate, 50>,`。
- **L1823 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Copyin>,`.
  **L1823 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Copyin>,`。
- **L1824 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L1824 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。

### Lines 1825-1856

````tablegen
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Linear>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_Shared>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_Message, 60>,
    VersionedClause<OMPC_NumThreads>,
    VersionedClause<OMPC_Order, 50>,
    VersionedClause<OMPC_Ordered>,
    VersionedClause<OMPC_ProcBind>,
    VersionedClause<OMPC_Schedule>,
    VersionedClause<OMPC_Severity, 60>,
  ];
  let leafConstructs = [OMP_Parallel, OMP_Do];
  let category = CA_Executable;
  let languages = [L_Fortran];
}
def OMP_ParallelDoSimd : Directive<[Spelling<"parallel do simd">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Aligned>,
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Copyin>,
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Linear>,
````
- **L1825 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L1825 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L1826 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L1826 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L1827 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Linear>,`.
  **L1827 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Linear>,`。
- **L1828 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L1828 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L1829 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L1829 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L1830 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L1830 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L1831 EN**: Executes a standalone statement or declaration: `];`.
  **L1831 CN**: 执行一条独立语句或声明：`];`。
- **L1832 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1832 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1833 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L1833 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L1834 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L1834 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L1835 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Message, 60>,`.
  **L1835 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Message, 60>,`。
- **L1836 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumThreads>,`.
  **L1836 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumThreads>,`。
- **L1837 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Order, 50>,`.
  **L1837 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Order, 50>,`。
- **L1838 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Ordered>,`.
  **L1838 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Ordered>,`。
- **L1839 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ProcBind>,`.
  **L1839 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ProcBind>,`。
- **L1840 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Schedule>,`.
  **L1840 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Schedule>,`。
- **L1841 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Severity, 60>,`.
  **L1841 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Severity, 60>,`。
- **L1842 EN**: Executes a standalone statement or declaration: `];`.
  **L1842 CN**: 执行一条独立语句或声明：`];`。
- **L1843 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1843 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1844 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1844 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1845 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1845 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1846 EN**: Closes the current lexical scope or compound statement.
  **L1846 CN**: 结束当前词法作用域或复合语句块。
- **L1847 EN**: Declares TableGen def `OMP_ParallelDoSimd`.
  **L1847 CN**: 声明 TableGen def `OMP_ParallelDoSimd`。
- **L1848 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1848 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1849 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Aligned>,`.
  **L1849 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Aligned>,`。
- **L1850 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L1850 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L1851 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Copyin>,`.
  **L1851 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Copyin>,`。
- **L1852 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L1852 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L1853 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L1853 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L1854 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L1854 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L1855 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L1855 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L1856 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Linear>,`.
  **L1856 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Linear>,`。

### Lines 1857-1888

````tablegen
    VersionedClause<OMPC_NonTemporal>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_Shared>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_Message, 60>,
    VersionedClause<OMPC_NumThreads>,
    VersionedClause<OMPC_Order, 50>,
    VersionedClause<OMPC_Ordered>,
    VersionedClause<OMPC_ProcBind>,
    VersionedClause<OMPC_SafeLen>,
    VersionedClause<OMPC_Schedule>,
    VersionedClause<OMPC_SimdLen>,
    VersionedClause<OMPC_Severity, 60>,
  ];
  let leafConstructs = [OMP_Parallel, OMP_Do, OMP_Simd];
  let category = CA_Executable;
  let languages = [L_Fortran];
}
def OMP_ParallelFor : Directive<[Spelling<"parallel for">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_Copyin>,
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Linear>,
    VersionedClause<OMPC_Message, 60>,
````
- **L1857 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NonTemporal>,`.
  **L1857 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NonTemporal>,`。
- **L1858 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L1858 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L1859 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L1859 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L1860 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L1860 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L1861 EN**: Executes a standalone statement or declaration: `];`.
  **L1861 CN**: 执行一条独立语句或声明：`];`。
- **L1862 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1862 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1863 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L1863 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L1864 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Message, 60>,`.
  **L1864 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Message, 60>,`。
- **L1865 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumThreads>,`.
  **L1865 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumThreads>,`。
- **L1866 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Order, 50>,`.
  **L1866 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Order, 50>,`。
- **L1867 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Ordered>,`.
  **L1867 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Ordered>,`。
- **L1868 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ProcBind>,`.
  **L1868 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ProcBind>,`。
- **L1869 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SafeLen>,`.
  **L1869 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SafeLen>,`。
- **L1870 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Schedule>,`.
  **L1870 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Schedule>,`。
- **L1871 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SimdLen>,`.
  **L1871 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SimdLen>,`。
- **L1872 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Severity, 60>,`.
  **L1872 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Severity, 60>,`。
- **L1873 EN**: Executes a standalone statement or declaration: `];`.
  **L1873 CN**: 执行一条独立语句或声明：`];`。
- **L1874 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1874 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1875 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1875 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1876 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1876 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1877 EN**: Closes the current lexical scope or compound statement.
  **L1877 CN**: 结束当前词法作用域或复合语句块。
- **L1878 EN**: Declares TableGen def `OMP_ParallelFor`.
  **L1878 CN**: 声明 TableGen def `OMP_ParallelFor`。
- **L1879 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1879 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1880 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L1880 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L1881 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L1881 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L1882 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Copyin>,`.
  **L1882 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Copyin>,`。
- **L1883 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L1883 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L1884 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L1884 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L1885 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L1885 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L1886 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L1886 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L1887 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Linear>,`.
  **L1887 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Linear>,`。
- **L1888 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Message, 60>,`.
  **L1888 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Message, 60>,`。

### Lines 1889-1920

````tablegen
    VersionedClause<OMPC_NumThreads>,
    VersionedClause<OMPC_OMPX_Attribute>,
    VersionedClause<OMPC_Order, 50>,
    VersionedClause<OMPC_Ordered>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_ProcBind>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_Schedule>,
    VersionedClause<OMPC_Severity, 60>,
    VersionedClause<OMPC_Shared>,
  ];
  let leafConstructs = [OMP_Parallel, OMP_For];
  let category = CA_Executable;
  let languages = [L_C];
}
def OMP_ParallelForSimd : Directive<[Spelling<"parallel for simd">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Aligned>,
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_Copyin>,
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Linear>,
    VersionedClause<OMPC_Message, 60>,
    VersionedClause<OMPC_NonTemporal, 50>,
    VersionedClause<OMPC_NumThreads>,
    VersionedClause<OMPC_OMPX_Attribute>,
    VersionedClause<OMPC_Order, 50>,
    VersionedClause<OMPC_Ordered>,
````
- **L1889 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumThreads>,`.
  **L1889 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumThreads>,`。
- **L1890 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_Attribute>,`.
  **L1890 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_Attribute>,`。
- **L1891 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Order, 50>,`.
  **L1891 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Order, 50>,`。
- **L1892 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Ordered>,`.
  **L1892 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Ordered>,`。
- **L1893 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L1893 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L1894 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ProcBind>,`.
  **L1894 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ProcBind>,`。
- **L1895 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L1895 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L1896 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Schedule>,`.
  **L1896 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Schedule>,`。
- **L1897 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Severity, 60>,`.
  **L1897 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Severity, 60>,`。
- **L1898 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L1898 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L1899 EN**: Executes a standalone statement or declaration: `];`.
  **L1899 CN**: 执行一条独立语句或声明：`];`。
- **L1900 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1900 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1901 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1901 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1902 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1902 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1903 EN**: Closes the current lexical scope or compound statement.
  **L1903 CN**: 结束当前词法作用域或复合语句块。
- **L1904 EN**: Declares TableGen def `OMP_ParallelForSimd`.
  **L1904 CN**: 声明 TableGen def `OMP_ParallelForSimd`。
- **L1905 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1905 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1906 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Aligned>,`.
  **L1906 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Aligned>,`。
- **L1907 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L1907 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L1908 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L1908 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L1909 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Copyin>,`.
  **L1909 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Copyin>,`。
- **L1910 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L1910 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L1911 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L1911 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L1912 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L1912 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L1913 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L1913 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L1914 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Linear>,`.
  **L1914 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Linear>,`。
- **L1915 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Message, 60>,`.
  **L1915 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Message, 60>,`。
- **L1916 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NonTemporal, 50>,`.
  **L1916 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NonTemporal, 50>,`。
- **L1917 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumThreads>,`.
  **L1917 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumThreads>,`。
- **L1918 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_Attribute>,`.
  **L1918 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_Attribute>,`。
- **L1919 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Order, 50>,`.
  **L1919 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Order, 50>,`。
- **L1920 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Ordered>,`.
  **L1920 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Ordered>,`。

### Lines 1921-1952

````tablegen
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_ProcBind>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_SafeLen>,
    VersionedClause<OMPC_Schedule>,
    VersionedClause<OMPC_Severity, 60>,
    VersionedClause<OMPC_Shared>,
    VersionedClause<OMPC_SimdLen>,
  ];
  let leafConstructs = [OMP_Parallel, OMP_For, OMP_Simd];
  let category = CA_Executable;
  let languages = [L_C];
}
def OMP_parallel_loop : Directive<[Spelling<"parallel loop">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Copyin>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_OMPX_Attribute>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_Shared>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_Bind, 50>,
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_Message, 60>,
    VersionedClause<OMPC_NumThreads>,
    VersionedClause<OMPC_Order>,
````
- **L1921 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L1921 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L1922 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ProcBind>,`.
  **L1922 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ProcBind>,`。
- **L1923 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L1923 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L1924 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SafeLen>,`.
  **L1924 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SafeLen>,`。
- **L1925 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Schedule>,`.
  **L1925 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Schedule>,`。
- **L1926 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Severity, 60>,`.
  **L1926 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Severity, 60>,`。
- **L1927 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L1927 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L1928 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SimdLen>,`.
  **L1928 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SimdLen>,`。
- **L1929 EN**: Executes a standalone statement or declaration: `];`.
  **L1929 CN**: 执行一条独立语句或声明：`];`。
- **L1930 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1930 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1931 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1931 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1932 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1932 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1933 EN**: Closes the current lexical scope or compound statement.
  **L1933 CN**: 结束当前词法作用域或复合语句块。
- **L1934 EN**: Declares TableGen def `OMP_parallel_loop`.
  **L1934 CN**: 声明 TableGen def `OMP_parallel_loop`。
- **L1935 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1935 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1936 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L1936 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L1937 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Copyin>,`.
  **L1937 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Copyin>,`。
- **L1938 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L1938 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L1939 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L1939 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L1940 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_Attribute>,`.
  **L1940 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_Attribute>,`。
- **L1941 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L1941 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L1942 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L1942 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L1943 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L1943 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L1944 EN**: Executes a standalone statement or declaration: `];`.
  **L1944 CN**: 执行一条独立语句或声明：`];`。
- **L1945 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1945 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1946 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Bind, 50>,`.
  **L1946 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Bind, 50>,`。
- **L1947 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L1947 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L1948 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L1948 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L1949 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L1949 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L1950 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Message, 60>,`.
  **L1950 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Message, 60>,`。
- **L1951 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumThreads>,`.
  **L1951 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumThreads>,`。
- **L1952 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Order>,`.
  **L1952 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Order>,`。

### Lines 1953-1984

````tablegen
    VersionedClause<OMPC_ProcBind>,
    VersionedClause<OMPC_Severity, 60>,
  ];
  let leafConstructs = [OMP_Parallel, OMP_loop];
  let category = CA_Executable;
}
def OMP_ParallelMasked : Directive<[Spelling<"parallel masked">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Copyin>,
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_Filter>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_Message, 60>,
    VersionedClause<OMPC_NumThreads>,
    VersionedClause<OMPC_OMPX_Attribute>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_ProcBind>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_Severity, 60>,
    VersionedClause<OMPC_Shared>,
  ];
  let leafConstructs = [OMP_Parallel, OMP_masked];
  let category = CA_Executable;
}
def OMP_ParallelMaskedTaskloop
    : Directive<[Spelling<"parallel masked taskloop">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_Copyin>,
````
- **L1953 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ProcBind>,`.
  **L1953 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ProcBind>,`。
- **L1954 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Severity, 60>,`.
  **L1954 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Severity, 60>,`。
- **L1955 EN**: Executes a standalone statement or declaration: `];`.
  **L1955 CN**: 执行一条独立语句或声明：`];`。
- **L1956 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1956 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1957 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1957 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1958 EN**: Closes the current lexical scope or compound statement.
  **L1958 CN**: 结束当前词法作用域或复合语句块。
- **L1959 EN**: Declares TableGen def `OMP_ParallelMasked`.
  **L1959 CN**: 声明 TableGen def `OMP_ParallelMasked`。
- **L1960 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1960 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1961 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L1961 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L1962 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Copyin>,`.
  **L1962 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Copyin>,`。
- **L1963 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L1963 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L1964 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Filter>,`.
  **L1964 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Filter>,`。
- **L1965 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L1965 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L1966 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L1966 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L1967 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Message, 60>,`.
  **L1967 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Message, 60>,`。
- **L1968 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumThreads>,`.
  **L1968 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumThreads>,`。
- **L1969 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_Attribute>,`.
  **L1969 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_Attribute>,`。
- **L1970 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L1970 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L1971 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ProcBind>,`.
  **L1971 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ProcBind>,`。
- **L1972 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L1972 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L1973 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Severity, 60>,`.
  **L1973 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Severity, 60>,`。
- **L1974 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L1974 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L1975 EN**: Executes a standalone statement or declaration: `];`.
  **L1975 CN**: 执行一条独立语句或声明：`];`。
- **L1976 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1976 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1977 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1977 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1978 EN**: Closes the current lexical scope or compound statement.
  **L1978 CN**: 结束当前词法作用域或复合语句块。
- **L1979 EN**: Declares TableGen def `OMP_ParallelMaskedTaskloop`.
  **L1979 CN**: 声明 TableGen def `OMP_ParallelMaskedTaskloop`。
- **L1980 EN**: Continues the surrounding expression or declaration: `: Directive<[Spelling<"parallel masked taskloop">]> {`.
  **L1980 CN**: 继续构造周围的表达式或声明：`: Directive<[Spelling<"parallel masked taskloop">]> {`。
- **L1981 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1981 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1982 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L1982 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L1983 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L1983 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L1984 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Copyin>,`.
  **L1984 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Copyin>,`。

### Lines 1985-2016

````tablegen
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_Filter>,
    VersionedClause<OMPC_Final>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_GrainSize>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Mergeable>,
    VersionedClause<OMPC_Message, 60>,
    VersionedClause<OMPC_NoGroup>,
    VersionedClause<OMPC_NumTasks>,
    VersionedClause<OMPC_NumThreads>,
    VersionedClause<OMPC_OMPX_Attribute>,
    VersionedClause<OMPC_Priority>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_ProcBind>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_Severity, 60>,
    VersionedClause<OMPC_Shared>,
    VersionedClause<OMPC_Untied>,
  ];
  let leafConstructs = [OMP_Parallel, OMP_masked, OMP_TaskLoop];
  let category = CA_Executable;
}
def OMP_ParallelMaskedTaskloopSimd
    : Directive<[Spelling<"parallel masked taskloop simd">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Aligned>,
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_Copyin>,
    VersionedClause<OMPC_Default>,
````
- **L1985 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L1985 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L1986 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Filter>,`.
  **L1986 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Filter>,`。
- **L1987 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Final>,`.
  **L1987 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Final>,`。
- **L1988 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L1988 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L1989 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_GrainSize>,`.
  **L1989 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_GrainSize>,`。
- **L1990 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L1990 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L1991 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L1991 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L1992 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Mergeable>,`.
  **L1992 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Mergeable>,`。
- **L1993 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Message, 60>,`.
  **L1993 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Message, 60>,`。
- **L1994 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoGroup>,`.
  **L1994 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoGroup>,`。
- **L1995 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumTasks>,`.
  **L1995 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumTasks>,`。
- **L1996 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumThreads>,`.
  **L1996 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumThreads>,`。
- **L1997 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_Attribute>,`.
  **L1997 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_Attribute>,`。
- **L1998 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Priority>,`.
  **L1998 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Priority>,`。
- **L1999 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L1999 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L2000 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ProcBind>,`.
  **L2000 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ProcBind>,`。
- **L2001 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L2001 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L2002 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Severity, 60>,`.
  **L2002 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Severity, 60>,`。
- **L2003 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L2003 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L2004 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Untied>,`.
  **L2004 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Untied>,`。
- **L2005 EN**: Executes a standalone statement or declaration: `];`.
  **L2005 CN**: 执行一条独立语句或声明：`];`。
- **L2006 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2006 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2007 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2007 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2008 EN**: Closes the current lexical scope or compound statement.
  **L2008 CN**: 结束当前词法作用域或复合语句块。
- **L2009 EN**: Declares TableGen def `OMP_ParallelMaskedTaskloopSimd`.
  **L2009 CN**: 声明 TableGen def `OMP_ParallelMaskedTaskloopSimd`。
- **L2010 EN**: Continues the surrounding expression or declaration: `: Directive<[Spelling<"parallel masked taskloop simd">]> {`.
  **L2010 CN**: 继续构造周围的表达式或声明：`: Directive<[Spelling<"parallel masked taskloop simd">]> {`。
- **L2011 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2011 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2012 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Aligned>,`.
  **L2012 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Aligned>,`。
- **L2013 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L2013 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L2014 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L2014 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L2015 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Copyin>,`.
  **L2015 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Copyin>,`。
- **L2016 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L2016 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。

### Lines 2017-2048

````tablegen
    VersionedClause<OMPC_Filter>,
    VersionedClause<OMPC_Final>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_GrainSize>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Linear>,
    VersionedClause<OMPC_Mergeable>,
    VersionedClause<OMPC_Message, 60>,
    VersionedClause<OMPC_NoGroup>,
    VersionedClause<OMPC_NonTemporal, 50>,
    VersionedClause<OMPC_NumTasks>,
    VersionedClause<OMPC_NumThreads>,
    VersionedClause<OMPC_OMPX_Attribute>,
    VersionedClause<OMPC_Order, 50>,
    VersionedClause<OMPC_Priority>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_ProcBind>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_SafeLen>,
    VersionedClause<OMPC_Severity, 60>,
    VersionedClause<OMPC_Shared>,
    VersionedClause<OMPC_SimdLen>,
    VersionedClause<OMPC_Untied>,
  ];
  let leafConstructs = [OMP_Parallel, OMP_masked, OMP_TaskLoop, OMP_Simd];
  let category = CA_Executable;
}
def OMP_ParallelMaster : Directive<[Spelling<"parallel master">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Copyin>,
````
- **L2017 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Filter>,`.
  **L2017 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Filter>,`。
- **L2018 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Final>,`.
  **L2018 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Final>,`。
- **L2019 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L2019 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L2020 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_GrainSize>,`.
  **L2020 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_GrainSize>,`。
- **L2021 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L2021 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L2022 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L2022 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L2023 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Linear>,`.
  **L2023 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Linear>,`。
- **L2024 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Mergeable>,`.
  **L2024 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Mergeable>,`。
- **L2025 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Message, 60>,`.
  **L2025 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Message, 60>,`。
- **L2026 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoGroup>,`.
  **L2026 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoGroup>,`。
- **L2027 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NonTemporal, 50>,`.
  **L2027 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NonTemporal, 50>,`。
- **L2028 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumTasks>,`.
  **L2028 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumTasks>,`。
- **L2029 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumThreads>,`.
  **L2029 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumThreads>,`。
- **L2030 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_Attribute>,`.
  **L2030 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_Attribute>,`。
- **L2031 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Order, 50>,`.
  **L2031 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Order, 50>,`。
- **L2032 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Priority>,`.
  **L2032 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Priority>,`。
- **L2033 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L2033 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L2034 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ProcBind>,`.
  **L2034 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ProcBind>,`。
- **L2035 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L2035 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L2036 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SafeLen>,`.
  **L2036 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SafeLen>,`。
- **L2037 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Severity, 60>,`.
  **L2037 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Severity, 60>,`。
- **L2038 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L2038 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L2039 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SimdLen>,`.
  **L2039 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SimdLen>,`。
- **L2040 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Untied>,`.
  **L2040 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Untied>,`。
- **L2041 EN**: Executes a standalone statement or declaration: `];`.
  **L2041 CN**: 执行一条独立语句或声明：`];`。
- **L2042 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2042 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2043 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2043 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2044 EN**: Closes the current lexical scope or compound statement.
  **L2044 CN**: 结束当前词法作用域或复合语句块。
- **L2045 EN**: Declares TableGen def `OMP_ParallelMaster`.
  **L2045 CN**: 声明 TableGen def `OMP_ParallelMaster`。
- **L2046 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2046 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2047 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L2047 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L2048 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Copyin>,`.
  **L2048 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Copyin>,`。

### Lines 2049-2080

````tablegen
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_Message, 60>,
    VersionedClause<OMPC_NumThreads>,
    VersionedClause<OMPC_OMPX_Attribute>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_ProcBind>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_Severity, 60>,
    VersionedClause<OMPC_Shared>,
  ];
  let leafConstructs = [OMP_Parallel, OMP_Master];
  let category = CA_Executable;
}
def OMP_ParallelMasterTaskloop
    : Directive<[Spelling<"parallel master taskloop">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_Copyin>,
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_Final>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_GrainSize>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Mergeable>,
    VersionedClause<OMPC_Message, 60>,
    VersionedClause<OMPC_NoGroup>,
    VersionedClause<OMPC_NumTasks>,
    VersionedClause<OMPC_NumThreads>,
````
- **L2049 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L2049 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L2050 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L2050 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L2051 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L2051 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L2052 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Message, 60>,`.
  **L2052 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Message, 60>,`。
- **L2053 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumThreads>,`.
  **L2053 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumThreads>,`。
- **L2054 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_Attribute>,`.
  **L2054 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_Attribute>,`。
- **L2055 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L2055 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L2056 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ProcBind>,`.
  **L2056 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ProcBind>,`。
- **L2057 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L2057 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L2058 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Severity, 60>,`.
  **L2058 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Severity, 60>,`。
- **L2059 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L2059 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L2060 EN**: Executes a standalone statement or declaration: `];`.
  **L2060 CN**: 执行一条独立语句或声明：`];`。
- **L2061 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2061 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2062 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2062 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2063 EN**: Closes the current lexical scope or compound statement.
  **L2063 CN**: 结束当前词法作用域或复合语句块。
- **L2064 EN**: Declares TableGen def `OMP_ParallelMasterTaskloop`.
  **L2064 CN**: 声明 TableGen def `OMP_ParallelMasterTaskloop`。
- **L2065 EN**: Continues the surrounding expression or declaration: `: Directive<[Spelling<"parallel master taskloop">]> {`.
  **L2065 CN**: 继续构造周围的表达式或声明：`: Directive<[Spelling<"parallel master taskloop">]> {`。
- **L2066 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2066 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2067 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L2067 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L2068 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L2068 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L2069 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Copyin>,`.
  **L2069 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Copyin>,`。
- **L2070 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L2070 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L2071 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Final>,`.
  **L2071 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Final>,`。
- **L2072 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L2072 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L2073 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_GrainSize>,`.
  **L2073 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_GrainSize>,`。
- **L2074 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L2074 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L2075 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L2075 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L2076 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Mergeable>,`.
  **L2076 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Mergeable>,`。
- **L2077 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Message, 60>,`.
  **L2077 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Message, 60>,`。
- **L2078 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoGroup>,`.
  **L2078 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoGroup>,`。
- **L2079 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumTasks>,`.
  **L2079 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumTasks>,`。
- **L2080 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumThreads>,`.
  **L2080 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumThreads>,`。

### Lines 2081-2112

````tablegen
    VersionedClause<OMPC_OMPX_Attribute>,
    VersionedClause<OMPC_Priority>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_ProcBind>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_Severity, 60>,
    VersionedClause<OMPC_Shared>,
    VersionedClause<OMPC_Untied>,
  ];
  let leafConstructs = [OMP_Parallel, OMP_Master, OMP_TaskLoop];
  let category = CA_Executable;
}
def OMP_ParallelMasterTaskloopSimd
    : Directive<[Spelling<"parallel master taskloop simd">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Aligned>,
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_Copyin>,
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_Final>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_GrainSize>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Linear>,
    VersionedClause<OMPC_Mergeable>,
    VersionedClause<OMPC_Message, 60>,
    VersionedClause<OMPC_NoGroup>,
    VersionedClause<OMPC_NonTemporal, 50>,
    VersionedClause<OMPC_NumTasks>,
    VersionedClause<OMPC_NumThreads>,
````
- **L2081 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_Attribute>,`.
  **L2081 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_Attribute>,`。
- **L2082 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Priority>,`.
  **L2082 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Priority>,`。
- **L2083 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L2083 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L2084 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ProcBind>,`.
  **L2084 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ProcBind>,`。
- **L2085 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L2085 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L2086 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Severity, 60>,`.
  **L2086 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Severity, 60>,`。
- **L2087 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L2087 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L2088 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Untied>,`.
  **L2088 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Untied>,`。
- **L2089 EN**: Executes a standalone statement or declaration: `];`.
  **L2089 CN**: 执行一条独立语句或声明：`];`。
- **L2090 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2090 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2091 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2091 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2092 EN**: Closes the current lexical scope or compound statement.
  **L2092 CN**: 结束当前词法作用域或复合语句块。
- **L2093 EN**: Declares TableGen def `OMP_ParallelMasterTaskloopSimd`.
  **L2093 CN**: 声明 TableGen def `OMP_ParallelMasterTaskloopSimd`。
- **L2094 EN**: Continues the surrounding expression or declaration: `: Directive<[Spelling<"parallel master taskloop simd">]> {`.
  **L2094 CN**: 继续构造周围的表达式或声明：`: Directive<[Spelling<"parallel master taskloop simd">]> {`。
- **L2095 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2095 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2096 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Aligned>,`.
  **L2096 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Aligned>,`。
- **L2097 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L2097 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L2098 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L2098 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L2099 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Copyin>,`.
  **L2099 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Copyin>,`。
- **L2100 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L2100 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L2101 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Final>,`.
  **L2101 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Final>,`。
- **L2102 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L2102 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L2103 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_GrainSize>,`.
  **L2103 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_GrainSize>,`。
- **L2104 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L2104 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L2105 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L2105 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L2106 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Linear>,`.
  **L2106 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Linear>,`。
- **L2107 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Mergeable>,`.
  **L2107 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Mergeable>,`。
- **L2108 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Message, 60>,`.
  **L2108 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Message, 60>,`。
- **L2109 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoGroup>,`.
  **L2109 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoGroup>,`。
- **L2110 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NonTemporal, 50>,`.
  **L2110 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NonTemporal, 50>,`。
- **L2111 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumTasks>,`.
  **L2111 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumTasks>,`。
- **L2112 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumThreads>,`.
  **L2112 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumThreads>,`。

### Lines 2113-2144

````tablegen
    VersionedClause<OMPC_OMPX_Attribute>,
    VersionedClause<OMPC_Order, 50>,
    VersionedClause<OMPC_Priority>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_ProcBind>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_SafeLen>,
    VersionedClause<OMPC_Severity, 60>,
    VersionedClause<OMPC_Shared>,
    VersionedClause<OMPC_SimdLen>,
    VersionedClause<OMPC_Untied>,
  ];
  let leafConstructs = [OMP_Parallel, OMP_Master, OMP_TaskLoop, OMP_Simd];
  let category = CA_Executable;
}
def OMP_ParallelSections : Directive<[Spelling<"parallel sections">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Copyin>,
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_OMPX_Attribute>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_ProcBind>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_Shared>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_Message, 60>,
    VersionedClause<OMPC_NumThreads>,
````
- **L2113 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_Attribute>,`.
  **L2113 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_Attribute>,`。
- **L2114 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Order, 50>,`.
  **L2114 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Order, 50>,`。
- **L2115 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Priority>,`.
  **L2115 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Priority>,`。
- **L2116 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L2116 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L2117 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ProcBind>,`.
  **L2117 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ProcBind>,`。
- **L2118 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L2118 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L2119 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SafeLen>,`.
  **L2119 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SafeLen>,`。
- **L2120 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Severity, 60>,`.
  **L2120 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Severity, 60>,`。
- **L2121 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L2121 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L2122 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SimdLen>,`.
  **L2122 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SimdLen>,`。
- **L2123 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Untied>,`.
  **L2123 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Untied>,`。
- **L2124 EN**: Executes a standalone statement or declaration: `];`.
  **L2124 CN**: 执行一条独立语句或声明：`];`。
- **L2125 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2125 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2126 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2126 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2127 EN**: Closes the current lexical scope or compound statement.
  **L2127 CN**: 结束当前词法作用域或复合语句块。
- **L2128 EN**: Declares TableGen def `OMP_ParallelSections`.
  **L2128 CN**: 声明 TableGen def `OMP_ParallelSections`。
- **L2129 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2129 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2130 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L2130 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L2131 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Copyin>,`.
  **L2131 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Copyin>,`。
- **L2132 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L2132 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L2133 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L2133 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L2134 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L2134 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L2135 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_Attribute>,`.
  **L2135 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_Attribute>,`。
- **L2136 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L2136 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L2137 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ProcBind>,`.
  **L2137 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ProcBind>,`。
- **L2138 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L2138 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L2139 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L2139 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L2140 EN**: Executes a standalone statement or declaration: `];`.
  **L2140 CN**: 执行一条独立语句或声明：`];`。
- **L2141 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2141 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2142 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L2142 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L2143 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Message, 60>,`.
  **L2143 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Message, 60>,`。
- **L2144 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumThreads>,`.
  **L2144 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumThreads>,`。

### Lines 2145-2176

````tablegen
    VersionedClause<OMPC_Severity, 60>,
  ];
  let leafConstructs = [OMP_Parallel, OMP_Sections];
  let category = CA_Executable;
}
def OMP_ParallelWorkshare : Directive<[Spelling<"parallel workshare">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Copyin>,
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_Shared>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_Message, 60>,
    VersionedClause<OMPC_NumThreads>,
    VersionedClause<OMPC_ProcBind>,
    VersionedClause<OMPC_Severity, 60>,
  ];
  let leafConstructs = [OMP_Parallel, OMP_Workshare];
  let category = CA_Executable;
  let languages = [L_Fortran];
}
def OMP_TargetParallel : Directive<[Spelling<"target parallel">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_Depend>,
    VersionedClause<OMPC_DynGroupprivate, 61>,
````
- **L2145 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Severity, 60>,`.
  **L2145 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Severity, 60>,`。
- **L2146 EN**: Executes a standalone statement or declaration: `];`.
  **L2146 CN**: 执行一条独立语句或声明：`];`。
- **L2147 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2147 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2148 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2148 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2149 EN**: Closes the current lexical scope or compound statement.
  **L2149 CN**: 结束当前词法作用域或复合语句块。
- **L2150 EN**: Declares TableGen def `OMP_ParallelWorkshare`.
  **L2150 CN**: 声明 TableGen def `OMP_ParallelWorkshare`。
- **L2151 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2151 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2152 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L2152 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L2153 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Copyin>,`.
  **L2153 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Copyin>,`。
- **L2154 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L2154 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L2155 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L2155 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L2156 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L2156 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L2157 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L2157 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L2158 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L2158 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L2159 EN**: Executes a standalone statement or declaration: `];`.
  **L2159 CN**: 执行一条独立语句或声明：`];`。
- **L2160 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2160 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2161 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L2161 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L2162 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Message, 60>,`.
  **L2162 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Message, 60>,`。
- **L2163 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumThreads>,`.
  **L2163 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumThreads>,`。
- **L2164 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ProcBind>,`.
  **L2164 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ProcBind>,`。
- **L2165 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Severity, 60>,`.
  **L2165 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Severity, 60>,`。
- **L2166 EN**: Executes a standalone statement or declaration: `];`.
  **L2166 CN**: 执行一条独立语句或声明：`];`。
- **L2167 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2167 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2168 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2168 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2169 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2169 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2170 EN**: Closes the current lexical scope or compound statement.
  **L2170 CN**: 结束当前词法作用域或复合语句块。
- **L2171 EN**: Declares TableGen def `OMP_TargetParallel`.
  **L2171 CN**: 声明 TableGen def `OMP_TargetParallel`。
- **L2172 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2172 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2173 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L2173 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L2174 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L2174 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L2175 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Depend>,`.
  **L2175 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Depend>,`。
- **L2176 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DynGroupprivate, 61>,`.
  **L2176 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DynGroupprivate, 61>,`。

### Lines 2177-2208

````tablegen
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_HasDeviceAddr, 51>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_IsDevicePtr>,
    VersionedClause<OMPC_Map>,
    VersionedClause<OMPC_NoWait>,
    VersionedClause<OMPC_OMPX_Attribute>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_Shared>,
    VersionedClause<OMPC_UsesAllocators, 50>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_DefaultMap>,
    VersionedClause<OMPC_Device>,
    VersionedClause<OMPC_Message, 60>,
    VersionedClause<OMPC_NumThreads>,
    VersionedClause<OMPC_OMPX_DynCGroupMem>,
    VersionedClause<OMPC_ProcBind>,
    VersionedClause<OMPC_Severity, 60>,
    VersionedClause<OMPC_ThreadLimit, 51>,
  ];
  let leafConstructs = [OMP_Target, OMP_Parallel];
  let category = CA_Executable;
}
def OMP_TargetParallelDo : Directive<[Spelling<"target parallel do">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Allocator>,
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_Depend>,
    VersionedClause<OMPC_DynGroupprivate, 61>,
    VersionedClause<OMPC_FirstPrivate>,
````
- **L2177 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L2177 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L2178 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_HasDeviceAddr, 51>,`.
  **L2178 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_HasDeviceAddr, 51>,`。
- **L2179 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L2179 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L2180 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_IsDevicePtr>,`.
  **L2180 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_IsDevicePtr>,`。
- **L2181 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Map>,`.
  **L2181 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Map>,`。
- **L2182 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoWait>,`.
  **L2182 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoWait>,`。
- **L2183 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_Attribute>,`.
  **L2183 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_Attribute>,`。
- **L2184 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L2184 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L2185 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L2185 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L2186 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L2186 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L2187 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_UsesAllocators, 50>,`.
  **L2187 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_UsesAllocators, 50>,`。
- **L2188 EN**: Executes a standalone statement or declaration: `];`.
  **L2188 CN**: 执行一条独立语句或声明：`];`。
- **L2189 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2189 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2190 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DefaultMap>,`.
  **L2190 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DefaultMap>,`。
- **L2191 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Device>,`.
  **L2191 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Device>,`。
- **L2192 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Message, 60>,`.
  **L2192 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Message, 60>,`。
- **L2193 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumThreads>,`.
  **L2193 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumThreads>,`。
- **L2194 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_DynCGroupMem>,`.
  **L2194 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_DynCGroupMem>,`。
- **L2195 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ProcBind>,`.
  **L2195 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ProcBind>,`。
- **L2196 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Severity, 60>,`.
  **L2196 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Severity, 60>,`。
- **L2197 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ThreadLimit, 51>,`.
  **L2197 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ThreadLimit, 51>,`。
- **L2198 EN**: Executes a standalone statement or declaration: `];`.
  **L2198 CN**: 执行一条独立语句或声明：`];`。
- **L2199 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2199 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2200 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2200 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2201 EN**: Closes the current lexical scope or compound statement.
  **L2201 CN**: 结束当前词法作用域或复合语句块。
- **L2202 EN**: Declares TableGen def `OMP_TargetParallelDo`.
  **L2202 CN**: 声明 TableGen def `OMP_TargetParallelDo`。
- **L2203 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2203 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2204 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocator>,`.
  **L2204 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocator>,`。
- **L2205 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L2205 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L2206 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Depend>,`.
  **L2206 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Depend>,`。
- **L2207 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DynGroupprivate, 61>,`.
  **L2207 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DynGroupprivate, 61>,`。
- **L2208 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L2208 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。

### Lines 2209-2240

````tablegen
    VersionedClause<OMPC_HasDeviceAddr, 51>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_IsDevicePtr>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Linear>,
    VersionedClause<OMPC_Map>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_Shared>,
    VersionedClause<OMPC_UsesAllocators>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_DefaultMap>,
    VersionedClause<OMPC_Device>,
    VersionedClause<OMPC_Message, 60>,
    VersionedClause<OMPC_NoWait>,
    VersionedClause<OMPC_NumThreads>,
    VersionedClause<OMPC_Order, 50>,
    VersionedClause<OMPC_Ordered>,
    VersionedClause<OMPC_ProcBind>,
    VersionedClause<OMPC_Schedule>,
    VersionedClause<OMPC_Severity, 60>,
  ];
  let leafConstructs = [OMP_Target, OMP_Parallel, OMP_Do];
  let category = CA_Executable;
  let languages = [L_Fortran];
}
def OMP_TargetParallelDoSimd
    : Directive<[Spelling<"target parallel do simd">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Aligned>,
````
- **L2209 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_HasDeviceAddr, 51>,`.
  **L2209 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_HasDeviceAddr, 51>,`。
- **L2210 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L2210 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L2211 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_IsDevicePtr>,`.
  **L2211 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_IsDevicePtr>,`。
- **L2212 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L2212 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L2213 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Linear>,`.
  **L2213 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Linear>,`。
- **L2214 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Map>,`.
  **L2214 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Map>,`。
- **L2215 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L2215 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L2216 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L2216 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L2217 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L2217 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L2218 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_UsesAllocators>,`.
  **L2218 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_UsesAllocators>,`。
- **L2219 EN**: Executes a standalone statement or declaration: `];`.
  **L2219 CN**: 执行一条独立语句或声明：`];`。
- **L2220 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2220 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2221 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L2221 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L2222 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DefaultMap>,`.
  **L2222 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DefaultMap>,`。
- **L2223 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Device>,`.
  **L2223 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Device>,`。
- **L2224 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Message, 60>,`.
  **L2224 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Message, 60>,`。
- **L2225 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoWait>,`.
  **L2225 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoWait>,`。
- **L2226 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumThreads>,`.
  **L2226 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumThreads>,`。
- **L2227 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Order, 50>,`.
  **L2227 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Order, 50>,`。
- **L2228 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Ordered>,`.
  **L2228 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Ordered>,`。
- **L2229 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ProcBind>,`.
  **L2229 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ProcBind>,`。
- **L2230 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Schedule>,`.
  **L2230 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Schedule>,`。
- **L2231 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Severity, 60>,`.
  **L2231 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Severity, 60>,`。
- **L2232 EN**: Executes a standalone statement or declaration: `];`.
  **L2232 CN**: 执行一条独立语句或声明：`];`。
- **L2233 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2233 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2234 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2234 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2235 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2235 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2236 EN**: Closes the current lexical scope or compound statement.
  **L2236 CN**: 结束当前词法作用域或复合语句块。
- **L2237 EN**: Declares TableGen def `OMP_TargetParallelDoSimd`.
  **L2237 CN**: 声明 TableGen def `OMP_TargetParallelDoSimd`。
- **L2238 EN**: Continues the surrounding expression or declaration: `: Directive<[Spelling<"target parallel do simd">]> {`.
  **L2238 CN**: 继续构造周围的表达式或声明：`: Directive<[Spelling<"target parallel do simd">]> {`。
- **L2239 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2239 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2240 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Aligned>,`.
  **L2240 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Aligned>,`。

### Lines 2241-2272

````tablegen
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_DefaultMap>,
    VersionedClause<OMPC_Depend>,
    VersionedClause<OMPC_Device>,
    VersionedClause<OMPC_DynGroupprivate, 61>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_HasDeviceAddr, 51>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_IsDevicePtr>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Linear>,
    VersionedClause<OMPC_Map>,
    VersionedClause<OMPC_Message, 60>,
    VersionedClause<OMPC_NonTemporal>,
    VersionedClause<OMPC_NoWait>,
    VersionedClause<OMPC_NumThreads>,
    VersionedClause<OMPC_Order, 50>,
    VersionedClause<OMPC_Ordered>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_ProcBind>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_SafeLen>,
    VersionedClause<OMPC_Schedule>,
    VersionedClause<OMPC_Severity, 60>,
    VersionedClause<OMPC_Shared>,
    VersionedClause<OMPC_SimdLen>,
    VersionedClause<OMPC_UsesAllocators>,
  ];
  let leafConstructs = [OMP_Target, OMP_Parallel, OMP_Do, OMP_Simd];
  let category = CA_Executable;
````
- **L2241 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L2241 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L2242 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L2242 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L2243 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L2243 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L2244 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DefaultMap>,`.
  **L2244 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DefaultMap>,`。
- **L2245 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Depend>,`.
  **L2245 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Depend>,`。
- **L2246 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Device>,`.
  **L2246 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Device>,`。
- **L2247 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DynGroupprivate, 61>,`.
  **L2247 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DynGroupprivate, 61>,`。
- **L2248 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L2248 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L2249 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_HasDeviceAddr, 51>,`.
  **L2249 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_HasDeviceAddr, 51>,`。
- **L2250 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L2250 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L2251 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_IsDevicePtr>,`.
  **L2251 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_IsDevicePtr>,`。
- **L2252 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L2252 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L2253 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Linear>,`.
  **L2253 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Linear>,`。
- **L2254 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Map>,`.
  **L2254 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Map>,`。
- **L2255 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Message, 60>,`.
  **L2255 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Message, 60>,`。
- **L2256 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NonTemporal>,`.
  **L2256 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NonTemporal>,`。
- **L2257 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoWait>,`.
  **L2257 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoWait>,`。
- **L2258 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumThreads>,`.
  **L2258 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumThreads>,`。
- **L2259 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Order, 50>,`.
  **L2259 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Order, 50>,`。
- **L2260 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Ordered>,`.
  **L2260 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Ordered>,`。
- **L2261 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L2261 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L2262 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ProcBind>,`.
  **L2262 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ProcBind>,`。
- **L2263 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L2263 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L2264 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SafeLen>,`.
  **L2264 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SafeLen>,`。
- **L2265 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Schedule>,`.
  **L2265 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Schedule>,`。
- **L2266 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Severity, 60>,`.
  **L2266 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Severity, 60>,`。
- **L2267 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L2267 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L2268 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SimdLen>,`.
  **L2268 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SimdLen>,`。
- **L2269 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_UsesAllocators>,`.
  **L2269 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_UsesAllocators>,`。
- **L2270 EN**: Executes a standalone statement or declaration: `];`.
  **L2270 CN**: 执行一条独立语句或声明：`];`。
- **L2271 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2271 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2272 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2272 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 2273-2304

````tablegen
  let languages = [L_Fortran];
}
def OMP_TargetParallelFor : Directive<[Spelling<"target parallel for">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_DefaultMap>,
    VersionedClause<OMPC_Depend>,
    VersionedClause<OMPC_Device>,
    VersionedClause<OMPC_DynGroupprivate, 61>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_HasDeviceAddr, 51>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_IsDevicePtr>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Linear>,
    VersionedClause<OMPC_Map>,
    VersionedClause<OMPC_Message, 60>,
    VersionedClause<OMPC_NoWait>,
    VersionedClause<OMPC_NumThreads>,
    VersionedClause<OMPC_OMPX_Attribute>,
    VersionedClause<OMPC_Order, 50>,
    VersionedClause<OMPC_Ordered>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_ProcBind>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_Schedule>,
    VersionedClause<OMPC_Severity, 60>,
    VersionedClause<OMPC_Shared>,
    VersionedClause<OMPC_UsesAllocators, 50>,
  ];
````
- **L2273 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2273 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2274 EN**: Closes the current lexical scope or compound statement.
  **L2274 CN**: 结束当前词法作用域或复合语句块。
- **L2275 EN**: Declares TableGen def `OMP_TargetParallelFor`.
  **L2275 CN**: 声明 TableGen def `OMP_TargetParallelFor`。
- **L2276 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2276 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2277 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L2277 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L2278 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L2278 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L2279 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L2279 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L2280 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DefaultMap>,`.
  **L2280 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DefaultMap>,`。
- **L2281 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Depend>,`.
  **L2281 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Depend>,`。
- **L2282 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Device>,`.
  **L2282 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Device>,`。
- **L2283 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DynGroupprivate, 61>,`.
  **L2283 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DynGroupprivate, 61>,`。
- **L2284 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L2284 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L2285 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_HasDeviceAddr, 51>,`.
  **L2285 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_HasDeviceAddr, 51>,`。
- **L2286 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L2286 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L2287 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_IsDevicePtr>,`.
  **L2287 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_IsDevicePtr>,`。
- **L2288 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L2288 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L2289 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Linear>,`.
  **L2289 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Linear>,`。
- **L2290 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Map>,`.
  **L2290 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Map>,`。
- **L2291 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Message, 60>,`.
  **L2291 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Message, 60>,`。
- **L2292 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoWait>,`.
  **L2292 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoWait>,`。
- **L2293 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumThreads>,`.
  **L2293 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumThreads>,`。
- **L2294 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_Attribute>,`.
  **L2294 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_Attribute>,`。
- **L2295 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Order, 50>,`.
  **L2295 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Order, 50>,`。
- **L2296 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Ordered>,`.
  **L2296 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Ordered>,`。
- **L2297 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L2297 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L2298 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ProcBind>,`.
  **L2298 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ProcBind>,`。
- **L2299 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L2299 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L2300 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Schedule>,`.
  **L2300 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Schedule>,`。
- **L2301 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Severity, 60>,`.
  **L2301 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Severity, 60>,`。
- **L2302 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L2302 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L2303 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_UsesAllocators, 50>,`.
  **L2303 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_UsesAllocators, 50>,`。
- **L2304 EN**: Executes a standalone statement or declaration: `];`.
  **L2304 CN**: 执行一条独立语句或声明：`];`。

### Lines 2305-2336

````tablegen
  let allowedOnceClauses = [
    VersionedClause<OMPC_OMPX_DynCGroupMem>,
    VersionedClause<OMPC_ThreadLimit, 51>,
  ];
  let leafConstructs = [OMP_Target, OMP_Parallel, OMP_For];
  let category = CA_Executable;
  let languages = [L_C];
}
def OMP_TargetParallelForSimd
    : Directive<[Spelling<"target parallel for simd">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Aligned>,
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_DefaultMap>,
    VersionedClause<OMPC_Depend>,
    VersionedClause<OMPC_Device>,
    VersionedClause<OMPC_DynGroupprivate, 61>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_HasDeviceAddr, 51>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_IsDevicePtr>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Linear>,
    VersionedClause<OMPC_Map>,
    VersionedClause<OMPC_Message, 60>,
    VersionedClause<OMPC_NonTemporal, 50>,
    VersionedClause<OMPC_NoWait>,
    VersionedClause<OMPC_NumThreads>,
    VersionedClause<OMPC_OMPX_Attribute>,
    VersionedClause<OMPC_Order, 50>,
````
- **L2305 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2305 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2306 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_DynCGroupMem>,`.
  **L2306 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_DynCGroupMem>,`。
- **L2307 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ThreadLimit, 51>,`.
  **L2307 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ThreadLimit, 51>,`。
- **L2308 EN**: Executes a standalone statement or declaration: `];`.
  **L2308 CN**: 执行一条独立语句或声明：`];`。
- **L2309 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2309 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2310 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2310 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2311 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2311 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2312 EN**: Closes the current lexical scope or compound statement.
  **L2312 CN**: 结束当前词法作用域或复合语句块。
- **L2313 EN**: Declares TableGen def `OMP_TargetParallelForSimd`.
  **L2313 CN**: 声明 TableGen def `OMP_TargetParallelForSimd`。
- **L2314 EN**: Continues the surrounding expression or declaration: `: Directive<[Spelling<"target parallel for simd">]> {`.
  **L2314 CN**: 继续构造周围的表达式或声明：`: Directive<[Spelling<"target parallel for simd">]> {`。
- **L2315 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2315 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2316 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Aligned>,`.
  **L2316 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Aligned>,`。
- **L2317 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L2317 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L2318 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L2318 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L2319 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L2319 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L2320 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DefaultMap>,`.
  **L2320 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DefaultMap>,`。
- **L2321 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Depend>,`.
  **L2321 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Depend>,`。
- **L2322 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Device>,`.
  **L2322 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Device>,`。
- **L2323 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DynGroupprivate, 61>,`.
  **L2323 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DynGroupprivate, 61>,`。
- **L2324 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L2324 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L2325 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_HasDeviceAddr, 51>,`.
  **L2325 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_HasDeviceAddr, 51>,`。
- **L2326 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L2326 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L2327 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_IsDevicePtr>,`.
  **L2327 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_IsDevicePtr>,`。
- **L2328 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L2328 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L2329 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Linear>,`.
  **L2329 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Linear>,`。
- **L2330 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Map>,`.
  **L2330 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Map>,`。
- **L2331 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Message, 60>,`.
  **L2331 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Message, 60>,`。
- **L2332 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NonTemporal, 50>,`.
  **L2332 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NonTemporal, 50>,`。
- **L2333 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoWait>,`.
  **L2333 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoWait>,`。
- **L2334 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumThreads>,`.
  **L2334 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumThreads>,`。
- **L2335 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_Attribute>,`.
  **L2335 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_Attribute>,`。
- **L2336 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Order, 50>,`.
  **L2336 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Order, 50>,`。

### Lines 2337-2368

````tablegen
    VersionedClause<OMPC_Ordered>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_ProcBind>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_SafeLen>,
    VersionedClause<OMPC_Schedule>,
    VersionedClause<OMPC_Severity, 60>,
    VersionedClause<OMPC_Shared>,
    VersionedClause<OMPC_SimdLen>,
    VersionedClause<OMPC_UsesAllocators, 50>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_OMPX_DynCGroupMem>,
    VersionedClause<OMPC_ThreadLimit, 51>,
  ];
  let leafConstructs = [OMP_Target, OMP_Parallel, OMP_For, OMP_Simd];
  let category = CA_Executable;
  let languages = [L_C];
}
def OMP_target_parallel_loop : Directive<[Spelling<"target parallel loop">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Depend>,
    VersionedClause<OMPC_Device>,
    VersionedClause<OMPC_DynGroupprivate, 61>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_HasDeviceAddr, 51>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_IsDevicePtr>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Map>,
    VersionedClause<OMPC_OMPX_Attribute>,
````
- **L2337 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Ordered>,`.
  **L2337 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Ordered>,`。
- **L2338 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L2338 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L2339 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ProcBind>,`.
  **L2339 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ProcBind>,`。
- **L2340 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L2340 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L2341 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SafeLen>,`.
  **L2341 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SafeLen>,`。
- **L2342 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Schedule>,`.
  **L2342 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Schedule>,`。
- **L2343 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Severity, 60>,`.
  **L2343 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Severity, 60>,`。
- **L2344 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L2344 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L2345 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SimdLen>,`.
  **L2345 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SimdLen>,`。
- **L2346 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_UsesAllocators, 50>,`.
  **L2346 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_UsesAllocators, 50>,`。
- **L2347 EN**: Executes a standalone statement or declaration: `];`.
  **L2347 CN**: 执行一条独立语句或声明：`];`。
- **L2348 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2348 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2349 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_DynCGroupMem>,`.
  **L2349 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_DynCGroupMem>,`。
- **L2350 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ThreadLimit, 51>,`.
  **L2350 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ThreadLimit, 51>,`。
- **L2351 EN**: Executes a standalone statement or declaration: `];`.
  **L2351 CN**: 执行一条独立语句或声明：`];`。
- **L2352 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2352 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2353 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2353 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2354 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2354 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2355 EN**: Closes the current lexical scope or compound statement.
  **L2355 CN**: 结束当前词法作用域或复合语句块。
- **L2356 EN**: Declares TableGen def `OMP_target_parallel_loop`.
  **L2356 CN**: 声明 TableGen def `OMP_target_parallel_loop`。
- **L2357 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2357 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2358 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L2358 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L2359 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Depend>,`.
  **L2359 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Depend>,`。
- **L2360 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Device>,`.
  **L2360 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Device>,`。
- **L2361 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DynGroupprivate, 61>,`.
  **L2361 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DynGroupprivate, 61>,`。
- **L2362 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L2362 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L2363 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_HasDeviceAddr, 51>,`.
  **L2363 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_HasDeviceAddr, 51>,`。
- **L2364 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L2364 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L2365 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_IsDevicePtr>,`.
  **L2365 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_IsDevicePtr>,`。
- **L2366 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L2366 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L2367 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Map>,`.
  **L2367 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Map>,`。
- **L2368 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_Attribute>,`.
  **L2368 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_Attribute>,`。

### Lines 2369-2400

````tablegen
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_Shared>,
    VersionedClause<OMPC_UsesAllocators, 50>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_Bind, 50>,
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_DefaultMap>,
    VersionedClause<OMPC_Message, 60>,
    VersionedClause<OMPC_NoWait>,
    VersionedClause<OMPC_NumThreads>,
    VersionedClause<OMPC_OMPX_DynCGroupMem>,
    VersionedClause<OMPC_Order>,
    VersionedClause<OMPC_ProcBind>,
    VersionedClause<OMPC_Severity, 60>,
    VersionedClause<OMPC_ThreadLimit, 51>,
  ];
  let leafConstructs = [OMP_Target, OMP_Parallel, OMP_loop];
  let category = CA_Executable;
}
def OMP_TargetSimd : Directive<[Spelling<"target simd">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Aligned>,
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Depend>,
    VersionedClause<OMPC_DynGroupprivate, 61>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_HasDeviceAddr, 51>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_IsDevicePtr>,
````
- **L2369 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L2369 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L2370 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L2370 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L2371 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L2371 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L2372 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_UsesAllocators, 50>,`.
  **L2372 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_UsesAllocators, 50>,`。
- **L2373 EN**: Executes a standalone statement or declaration: `];`.
  **L2373 CN**: 执行一条独立语句或声明：`];`。
- **L2374 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2374 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2375 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Bind, 50>,`.
  **L2375 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Bind, 50>,`。
- **L2376 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L2376 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L2377 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L2377 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L2378 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DefaultMap>,`.
  **L2378 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DefaultMap>,`。
- **L2379 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Message, 60>,`.
  **L2379 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Message, 60>,`。
- **L2380 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoWait>,`.
  **L2380 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoWait>,`。
- **L2381 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumThreads>,`.
  **L2381 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumThreads>,`。
- **L2382 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_DynCGroupMem>,`.
  **L2382 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_DynCGroupMem>,`。
- **L2383 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Order>,`.
  **L2383 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Order>,`。
- **L2384 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ProcBind>,`.
  **L2384 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ProcBind>,`。
- **L2385 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Severity, 60>,`.
  **L2385 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Severity, 60>,`。
- **L2386 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ThreadLimit, 51>,`.
  **L2386 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ThreadLimit, 51>,`。
- **L2387 EN**: Executes a standalone statement or declaration: `];`.
  **L2387 CN**: 执行一条独立语句或声明：`];`。
- **L2388 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2388 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2389 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2389 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2390 EN**: Closes the current lexical scope or compound statement.
  **L2390 CN**: 结束当前词法作用域或复合语句块。
- **L2391 EN**: Declares TableGen def `OMP_TargetSimd`.
  **L2391 CN**: 声明 TableGen def `OMP_TargetSimd`。
- **L2392 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2392 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2393 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Aligned>,`.
  **L2393 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Aligned>,`。
- **L2394 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L2394 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L2395 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Depend>,`.
  **L2395 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Depend>,`。
- **L2396 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DynGroupprivate, 61>,`.
  **L2396 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DynGroupprivate, 61>,`。
- **L2397 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L2397 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L2398 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_HasDeviceAddr, 51>,`.
  **L2398 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_HasDeviceAddr, 51>,`。
- **L2399 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L2399 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L2400 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_IsDevicePtr>,`.
  **L2400 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_IsDevicePtr>,`。

### Lines 2401-2432

````tablegen
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Linear>,
    VersionedClause<OMPC_Map>,
    VersionedClause<OMPC_NonTemporal, 50>,
    VersionedClause<OMPC_NoWait>,
    VersionedClause<OMPC_OMPX_Attribute>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_Shared>,
    VersionedClause<OMPC_UsesAllocators, 50>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_DefaultMap>,
    VersionedClause<OMPC_Device>,
    VersionedClause<OMPC_Message, 60>,
    VersionedClause<OMPC_NumThreads>,
    VersionedClause<OMPC_OMPX_DynCGroupMem>,
    VersionedClause<OMPC_Order, 50>,
    VersionedClause<OMPC_ProcBind>,
    VersionedClause<OMPC_SafeLen>,
    VersionedClause<OMPC_Schedule>,
    VersionedClause<OMPC_Severity, 60>,
    VersionedClause<OMPC_SimdLen>,
    VersionedClause<OMPC_ThreadLimit, 51>,
  ];
  let leafConstructs = [OMP_Target, OMP_Simd];
  let category = CA_Executable;
}
def OMP_TargetTeams : Directive<[Spelling<"target teams">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Allocate>,
````
- **L2401 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L2401 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L2402 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Linear>,`.
  **L2402 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Linear>,`。
- **L2403 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Map>,`.
  **L2403 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Map>,`。
- **L2404 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NonTemporal, 50>,`.
  **L2404 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NonTemporal, 50>,`。
- **L2405 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoWait>,`.
  **L2405 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoWait>,`。
- **L2406 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_Attribute>,`.
  **L2406 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_Attribute>,`。
- **L2407 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L2407 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L2408 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L2408 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L2409 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L2409 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L2410 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_UsesAllocators, 50>,`.
  **L2410 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_UsesAllocators, 50>,`。
- **L2411 EN**: Executes a standalone statement or declaration: `];`.
  **L2411 CN**: 执行一条独立语句或声明：`];`。
- **L2412 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2412 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2413 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L2413 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L2414 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DefaultMap>,`.
  **L2414 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DefaultMap>,`。
- **L2415 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Device>,`.
  **L2415 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Device>,`。
- **L2416 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Message, 60>,`.
  **L2416 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Message, 60>,`。
- **L2417 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumThreads>,`.
  **L2417 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumThreads>,`。
- **L2418 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_DynCGroupMem>,`.
  **L2418 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_DynCGroupMem>,`。
- **L2419 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Order, 50>,`.
  **L2419 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Order, 50>,`。
- **L2420 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ProcBind>,`.
  **L2420 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ProcBind>,`。
- **L2421 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SafeLen>,`.
  **L2421 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SafeLen>,`。
- **L2422 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Schedule>,`.
  **L2422 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Schedule>,`。
- **L2423 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Severity, 60>,`.
  **L2423 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Severity, 60>,`。
- **L2424 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SimdLen>,`.
  **L2424 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SimdLen>,`。
- **L2425 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ThreadLimit, 51>,`.
  **L2425 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ThreadLimit, 51>,`。
- **L2426 EN**: Executes a standalone statement or declaration: `];`.
  **L2426 CN**: 执行一条独立语句或声明：`];`。
- **L2427 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2427 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2428 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2428 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2429 EN**: Closes the current lexical scope or compound statement.
  **L2429 CN**: 结束当前词法作用域或复合语句块。
- **L2430 EN**: Declares TableGen def `OMP_TargetTeams`.
  **L2430 CN**: 声明 TableGen def `OMP_TargetTeams`。
- **L2431 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2431 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2432 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L2432 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。

### Lines 2433-2464

````tablegen
    VersionedClause<OMPC_Depend>,
    VersionedClause<OMPC_DynGroupprivate, 61>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_HasDeviceAddr, 51>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_IsDevicePtr>,
    VersionedClause<OMPC_Map>,
    VersionedClause<OMPC_OMPX_Attribute>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_Shared>,
    VersionedClause<OMPC_UsesAllocators, 50>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_DefaultMap>,
    VersionedClause<OMPC_Device>,
    VersionedClause<OMPC_NoWait>,
    VersionedClause<OMPC_NumTeams>,
    VersionedClause<OMPC_OMPX_DynCGroupMem>,
    VersionedClause<OMPC_OMPX_Bare>,
    VersionedClause<OMPC_ThreadLimit>,
  ];
  let leafConstructs = [OMP_Target, OMP_Teams];
  let category = CA_Executable;
}
def OMP_TargetTeamsDistribute
    : Directive<[Spelling<"target teams distribute">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Depend>,
    VersionedClause<OMPC_DynGroupprivate, 61>,
````
- **L2433 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Depend>,`.
  **L2433 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Depend>,`。
- **L2434 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DynGroupprivate, 61>,`.
  **L2434 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DynGroupprivate, 61>,`。
- **L2435 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L2435 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L2436 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_HasDeviceAddr, 51>,`.
  **L2436 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_HasDeviceAddr, 51>,`。
- **L2437 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L2437 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L2438 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_IsDevicePtr>,`.
  **L2438 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_IsDevicePtr>,`。
- **L2439 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Map>,`.
  **L2439 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Map>,`。
- **L2440 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_Attribute>,`.
  **L2440 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_Attribute>,`。
- **L2441 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L2441 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L2442 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L2442 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L2443 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L2443 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L2444 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_UsesAllocators, 50>,`.
  **L2444 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_UsesAllocators, 50>,`。
- **L2445 EN**: Executes a standalone statement or declaration: `];`.
  **L2445 CN**: 执行一条独立语句或声明：`];`。
- **L2446 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2446 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2447 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L2447 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L2448 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DefaultMap>,`.
  **L2448 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DefaultMap>,`。
- **L2449 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Device>,`.
  **L2449 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Device>,`。
- **L2450 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoWait>,`.
  **L2450 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoWait>,`。
- **L2451 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumTeams>,`.
  **L2451 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumTeams>,`。
- **L2452 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_DynCGroupMem>,`.
  **L2452 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_DynCGroupMem>,`。
- **L2453 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_Bare>,`.
  **L2453 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_Bare>,`。
- **L2454 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ThreadLimit>,`.
  **L2454 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ThreadLimit>,`。
- **L2455 EN**: Executes a standalone statement or declaration: `];`.
  **L2455 CN**: 执行一条独立语句或声明：`];`。
- **L2456 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2456 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2457 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2457 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2458 EN**: Closes the current lexical scope or compound statement.
  **L2458 CN**: 结束当前词法作用域或复合语句块。
- **L2459 EN**: Declares TableGen def `OMP_TargetTeamsDistribute`.
  **L2459 CN**: 声明 TableGen def `OMP_TargetTeamsDistribute`。
- **L2460 EN**: Continues the surrounding expression or declaration: `: Directive<[Spelling<"target teams distribute">]> {`.
  **L2460 CN**: 继续构造周围的表达式或声明：`: Directive<[Spelling<"target teams distribute">]> {`。
- **L2461 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2461 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2462 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L2462 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L2463 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Depend>,`.
  **L2463 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Depend>,`。
- **L2464 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DynGroupprivate, 61>,`.
  **L2464 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DynGroupprivate, 61>,`。

### Lines 2465-2496

````tablegen
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_HasDeviceAddr, 51>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_IsDevicePtr>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Map>,
    VersionedClause<OMPC_OMPX_Attribute>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_Shared>,
    VersionedClause<OMPC_UsesAllocators, 50>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_DefaultMap>,
    VersionedClause<OMPC_Device>,
    VersionedClause<OMPC_DistSchedule>,
    VersionedClause<OMPC_NoWait>,
    VersionedClause<OMPC_NumTeams>,
    VersionedClause<OMPC_OMPX_DynCGroupMem>,
    VersionedClause<OMPC_Order, 50>,
    VersionedClause<OMPC_ThreadLimit>,
  ];
  let leafConstructs = [OMP_Target, OMP_Teams, OMP_Distribute];
  let category = CA_Executable;
}
def OMP_TargetTeamsDistributeParallelDo
    : Directive<[Spelling<"target teams distribute parallel do">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Depend>,
````
- **L2465 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L2465 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L2466 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_HasDeviceAddr, 51>,`.
  **L2466 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_HasDeviceAddr, 51>,`。
- **L2467 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L2467 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L2468 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_IsDevicePtr>,`.
  **L2468 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_IsDevicePtr>,`。
- **L2469 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L2469 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L2470 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Map>,`.
  **L2470 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Map>,`。
- **L2471 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_Attribute>,`.
  **L2471 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_Attribute>,`。
- **L2472 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L2472 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L2473 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L2473 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L2474 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L2474 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L2475 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_UsesAllocators, 50>,`.
  **L2475 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_UsesAllocators, 50>,`。
- **L2476 EN**: Executes a standalone statement or declaration: `];`.
  **L2476 CN**: 执行一条独立语句或声明：`];`。
- **L2477 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2477 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2478 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L2478 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L2479 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L2479 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L2480 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DefaultMap>,`.
  **L2480 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DefaultMap>,`。
- **L2481 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Device>,`.
  **L2481 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Device>,`。
- **L2482 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DistSchedule>,`.
  **L2482 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DistSchedule>,`。
- **L2483 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoWait>,`.
  **L2483 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoWait>,`。
- **L2484 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumTeams>,`.
  **L2484 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumTeams>,`。
- **L2485 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_DynCGroupMem>,`.
  **L2485 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_DynCGroupMem>,`。
- **L2486 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Order, 50>,`.
  **L2486 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Order, 50>,`。
- **L2487 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ThreadLimit>,`.
  **L2487 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ThreadLimit>,`。
- **L2488 EN**: Executes a standalone statement or declaration: `];`.
  **L2488 CN**: 执行一条独立语句或声明：`];`。
- **L2489 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2489 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2490 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2490 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2491 EN**: Closes the current lexical scope or compound statement.
  **L2491 CN**: 结束当前词法作用域或复合语句块。
- **L2492 EN**: Declares TableGen def `OMP_TargetTeamsDistributeParallelDo`.
  **L2492 CN**: 声明 TableGen def `OMP_TargetTeamsDistributeParallelDo`。
- **L2493 EN**: Continues the surrounding expression or declaration: `: Directive<[Spelling<"target teams distribute parallel do">]> {`.
  **L2493 CN**: 继续构造周围的表达式或声明：`: Directive<[Spelling<"target teams distribute parallel do">]> {`。
- **L2494 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2494 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2495 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L2495 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L2496 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Depend>,`.
  **L2496 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Depend>,`。

### Lines 2497-2528

````tablegen
    VersionedClause<OMPC_DynGroupprivate, 61>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_HasDeviceAddr, 51>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_IsDevicePtr>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Linear>,
    VersionedClause<OMPC_Map>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_Shared>,
    VersionedClause<OMPC_UsesAllocators>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_DefaultMap>,
    VersionedClause<OMPC_Device>,
    VersionedClause<OMPC_DistSchedule>,
    VersionedClause<OMPC_Message, 60>,
    VersionedClause<OMPC_NoWait>,
    VersionedClause<OMPC_NumTeams>,
    VersionedClause<OMPC_NumThreads>,
    VersionedClause<OMPC_Order, 50>,
    VersionedClause<OMPC_ProcBind>,
    VersionedClause<OMPC_Schedule>,
    VersionedClause<OMPC_Severity, 60>,
    VersionedClause<OMPC_ThreadLimit>,
  ];
  let leafConstructs =
      [OMP_Target, OMP_Teams, OMP_Distribute, OMP_Parallel, OMP_Do];
  let category = CA_Executable;
````
- **L2497 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DynGroupprivate, 61>,`.
  **L2497 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DynGroupprivate, 61>,`。
- **L2498 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L2498 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L2499 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_HasDeviceAddr, 51>,`.
  **L2499 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_HasDeviceAddr, 51>,`。
- **L2500 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L2500 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L2501 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_IsDevicePtr>,`.
  **L2501 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_IsDevicePtr>,`。
- **L2502 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L2502 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L2503 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Linear>,`.
  **L2503 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Linear>,`。
- **L2504 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Map>,`.
  **L2504 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Map>,`。
- **L2505 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L2505 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L2506 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L2506 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L2507 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L2507 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L2508 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_UsesAllocators>,`.
  **L2508 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_UsesAllocators>,`。
- **L2509 EN**: Executes a standalone statement or declaration: `];`.
  **L2509 CN**: 执行一条独立语句或声明：`];`。
- **L2510 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2510 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2511 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L2511 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L2512 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L2512 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L2513 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DefaultMap>,`.
  **L2513 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DefaultMap>,`。
- **L2514 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Device>,`.
  **L2514 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Device>,`。
- **L2515 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DistSchedule>,`.
  **L2515 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DistSchedule>,`。
- **L2516 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Message, 60>,`.
  **L2516 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Message, 60>,`。
- **L2517 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoWait>,`.
  **L2517 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoWait>,`。
- **L2518 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumTeams>,`.
  **L2518 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumTeams>,`。
- **L2519 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumThreads>,`.
  **L2519 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumThreads>,`。
- **L2520 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Order, 50>,`.
  **L2520 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Order, 50>,`。
- **L2521 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ProcBind>,`.
  **L2521 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ProcBind>,`。
- **L2522 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Schedule>,`.
  **L2522 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Schedule>,`。
- **L2523 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Severity, 60>,`.
  **L2523 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Severity, 60>,`。
- **L2524 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ThreadLimit>,`.
  **L2524 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ThreadLimit>,`。
- **L2525 EN**: Executes a standalone statement or declaration: `];`.
  **L2525 CN**: 执行一条独立语句或声明：`];`。
- **L2526 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2526 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2527 EN**: Executes a standalone statement or declaration: `[OMP_Target, OMP_Teams, OMP_Distribute, OMP_Parallel, OMP_Do];`.
  **L2527 CN**: 执行一条独立语句或声明：`[OMP_Target, OMP_Teams, OMP_Distribute, OMP_Parallel, OMP_Do];`。
- **L2528 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2528 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 2529-2560

````tablegen
  let languages = [L_Fortran];
}
def OMP_TargetTeamsDistributeParallelDoSimd
    : Directive<[Spelling<"target teams distribute parallel do simd">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Aligned>,
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Depend>,
    VersionedClause<OMPC_DynGroupprivate, 61>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_HasDeviceAddr, 51>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_IsDevicePtr>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Linear>,
    VersionedClause<OMPC_Map>,
    VersionedClause<OMPC_NonTemporal>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_Shared>,
    VersionedClause<OMPC_UsesAllocators>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_DefaultMap>,
    VersionedClause<OMPC_Device>,
    VersionedClause<OMPC_DistSchedule>,
    VersionedClause<OMPC_Message, 60>,
    VersionedClause<OMPC_NoWait>,
    VersionedClause<OMPC_NumTeams>,
    VersionedClause<OMPC_NumThreads>,
````
- **L2529 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2529 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2530 EN**: Closes the current lexical scope or compound statement.
  **L2530 CN**: 结束当前词法作用域或复合语句块。
- **L2531 EN**: Declares TableGen def `OMP_TargetTeamsDistributeParallelDoSimd`.
  **L2531 CN**: 声明 TableGen def `OMP_TargetTeamsDistributeParallelDoSimd`。
- **L2532 EN**: Continues the surrounding expression or declaration: `: Directive<[Spelling<"target teams distribute parallel do simd">]> {`.
  **L2532 CN**: 继续构造周围的表达式或声明：`: Directive<[Spelling<"target teams distribute parallel do simd">]> {`。
- **L2533 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2533 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2534 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Aligned>,`.
  **L2534 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Aligned>,`。
- **L2535 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L2535 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L2536 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Depend>,`.
  **L2536 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Depend>,`。
- **L2537 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DynGroupprivate, 61>,`.
  **L2537 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DynGroupprivate, 61>,`。
- **L2538 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L2538 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L2539 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_HasDeviceAddr, 51>,`.
  **L2539 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_HasDeviceAddr, 51>,`。
- **L2540 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L2540 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L2541 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_IsDevicePtr>,`.
  **L2541 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_IsDevicePtr>,`。
- **L2542 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L2542 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L2543 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Linear>,`.
  **L2543 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Linear>,`。
- **L2544 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Map>,`.
  **L2544 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Map>,`。
- **L2545 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NonTemporal>,`.
  **L2545 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NonTemporal>,`。
- **L2546 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L2546 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L2547 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L2547 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L2548 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L2548 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L2549 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_UsesAllocators>,`.
  **L2549 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_UsesAllocators>,`。
- **L2550 EN**: Executes a standalone statement or declaration: `];`.
  **L2550 CN**: 执行一条独立语句或声明：`];`。
- **L2551 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2551 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2552 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L2552 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L2553 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L2553 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L2554 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DefaultMap>,`.
  **L2554 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DefaultMap>,`。
- **L2555 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Device>,`.
  **L2555 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Device>,`。
- **L2556 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DistSchedule>,`.
  **L2556 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DistSchedule>,`。
- **L2557 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Message, 60>,`.
  **L2557 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Message, 60>,`。
- **L2558 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoWait>,`.
  **L2558 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoWait>,`。
- **L2559 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumTeams>,`.
  **L2559 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumTeams>,`。
- **L2560 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumThreads>,`.
  **L2560 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumThreads>,`。

### Lines 2561-2592

````tablegen
    VersionedClause<OMPC_Order, 50>,
    VersionedClause<OMPC_ProcBind>,
    VersionedClause<OMPC_SafeLen>,
    VersionedClause<OMPC_Schedule>,
    VersionedClause<OMPC_Severity, 60>,
    VersionedClause<OMPC_SimdLen>,
    VersionedClause<OMPC_ThreadLimit>,
  ];
  let leafConstructs =
      [OMP_Target, OMP_Teams, OMP_Distribute, OMP_Parallel, OMP_Do, OMP_Simd];
  let category = CA_Executable;
  let languages = [L_Fortran];
}
def OMP_TargetTeamsDistributeParallelFor
    : Directive<[Spelling<"target teams distribute parallel for">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_DefaultMap>,
    VersionedClause<OMPC_Depend>,
    VersionedClause<OMPC_Device>,
    VersionedClause<OMPC_DistSchedule>,
    VersionedClause<OMPC_DynGroupprivate, 61>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_HasDeviceAddr, 51>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_IsDevicePtr>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Map>,
    VersionedClause<OMPC_Message, 60>,
    VersionedClause<OMPC_NoWait>,
````
- **L2561 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Order, 50>,`.
  **L2561 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Order, 50>,`。
- **L2562 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ProcBind>,`.
  **L2562 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ProcBind>,`。
- **L2563 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SafeLen>,`.
  **L2563 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SafeLen>,`。
- **L2564 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Schedule>,`.
  **L2564 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Schedule>,`。
- **L2565 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Severity, 60>,`.
  **L2565 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Severity, 60>,`。
- **L2566 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SimdLen>,`.
  **L2566 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SimdLen>,`。
- **L2567 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ThreadLimit>,`.
  **L2567 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ThreadLimit>,`。
- **L2568 EN**: Executes a standalone statement or declaration: `];`.
  **L2568 CN**: 执行一条独立语句或声明：`];`。
- **L2569 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2569 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2570 EN**: Executes a standalone statement or declaration: `[OMP_Target, OMP_Teams, OMP_Distribute, OMP_Parallel, OMP_Do, OMP_Simd];`.
  **L2570 CN**: 执行一条独立语句或声明：`[OMP_Target, OMP_Teams, OMP_Distribute, OMP_Parallel, OMP_Do, OMP_Simd];`。
- **L2571 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2571 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2572 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2572 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2573 EN**: Closes the current lexical scope or compound statement.
  **L2573 CN**: 结束当前词法作用域或复合语句块。
- **L2574 EN**: Declares TableGen def `OMP_TargetTeamsDistributeParallelFor`.
  **L2574 CN**: 声明 TableGen def `OMP_TargetTeamsDistributeParallelFor`。
- **L2575 EN**: Continues the surrounding expression or declaration: `: Directive<[Spelling<"target teams distribute parallel for">]> {`.
  **L2575 CN**: 继续构造周围的表达式或声明：`: Directive<[Spelling<"target teams distribute parallel for">]> {`。
- **L2576 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2576 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2577 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L2577 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L2578 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L2578 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L2579 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L2579 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L2580 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DefaultMap>,`.
  **L2580 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DefaultMap>,`。
- **L2581 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Depend>,`.
  **L2581 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Depend>,`。
- **L2582 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Device>,`.
  **L2582 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Device>,`。
- **L2583 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DistSchedule>,`.
  **L2583 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DistSchedule>,`。
- **L2584 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DynGroupprivate, 61>,`.
  **L2584 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DynGroupprivate, 61>,`。
- **L2585 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L2585 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L2586 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_HasDeviceAddr, 51>,`.
  **L2586 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_HasDeviceAddr, 51>,`。
- **L2587 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L2587 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L2588 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_IsDevicePtr>,`.
  **L2588 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_IsDevicePtr>,`。
- **L2589 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L2589 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L2590 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Map>,`.
  **L2590 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Map>,`。
- **L2591 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Message, 60>,`.
  **L2591 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Message, 60>,`。
- **L2592 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoWait>,`.
  **L2592 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoWait>,`。

### Lines 2593-2624

````tablegen
    VersionedClause<OMPC_NumTeams>,
    VersionedClause<OMPC_NumThreads>,
    VersionedClause<OMPC_OMPX_Attribute>,
    VersionedClause<OMPC_Order, 50>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_ProcBind>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_Schedule>,
    VersionedClause<OMPC_Severity, 60>,
    VersionedClause<OMPC_Shared>,
    VersionedClause<OMPC_ThreadLimit>,
    VersionedClause<OMPC_UsesAllocators, 50>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_OMPX_DynCGroupMem>,
  ];
  let leafConstructs =
      [OMP_Target, OMP_Teams, OMP_Distribute, OMP_Parallel, OMP_For];
  let category = CA_Executable;
  let languages = [L_C];
}
def OMP_TargetTeamsDistributeParallelForSimd
    : Directive<[Spelling<"target teams distribute parallel for simd">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Aligned>,
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_DefaultMap>,
    VersionedClause<OMPC_Depend>,
    VersionedClause<OMPC_Device>,
    VersionedClause<OMPC_DistSchedule>,
````
- **L2593 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumTeams>,`.
  **L2593 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumTeams>,`。
- **L2594 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumThreads>,`.
  **L2594 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumThreads>,`。
- **L2595 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_Attribute>,`.
  **L2595 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_Attribute>,`。
- **L2596 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Order, 50>,`.
  **L2596 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Order, 50>,`。
- **L2597 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L2597 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L2598 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ProcBind>,`.
  **L2598 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ProcBind>,`。
- **L2599 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L2599 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L2600 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Schedule>,`.
  **L2600 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Schedule>,`。
- **L2601 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Severity, 60>,`.
  **L2601 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Severity, 60>,`。
- **L2602 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L2602 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L2603 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ThreadLimit>,`.
  **L2603 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ThreadLimit>,`。
- **L2604 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_UsesAllocators, 50>,`.
  **L2604 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_UsesAllocators, 50>,`。
- **L2605 EN**: Executes a standalone statement or declaration: `];`.
  **L2605 CN**: 执行一条独立语句或声明：`];`。
- **L2606 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2606 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2607 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_DynCGroupMem>,`.
  **L2607 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_DynCGroupMem>,`。
- **L2608 EN**: Executes a standalone statement or declaration: `];`.
  **L2608 CN**: 执行一条独立语句或声明：`];`。
- **L2609 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2609 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2610 EN**: Executes a standalone statement or declaration: `[OMP_Target, OMP_Teams, OMP_Distribute, OMP_Parallel, OMP_For];`.
  **L2610 CN**: 执行一条独立语句或声明：`[OMP_Target, OMP_Teams, OMP_Distribute, OMP_Parallel, OMP_For];`。
- **L2611 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2611 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2612 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2612 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2613 EN**: Closes the current lexical scope or compound statement.
  **L2613 CN**: 结束当前词法作用域或复合语句块。
- **L2614 EN**: Declares TableGen def `OMP_TargetTeamsDistributeParallelForSimd`.
  **L2614 CN**: 声明 TableGen def `OMP_TargetTeamsDistributeParallelForSimd`。
- **L2615 EN**: Continues the surrounding expression or declaration: `: Directive<[Spelling<"target teams distribute parallel for simd">]> {`.
  **L2615 CN**: 继续构造周围的表达式或声明：`: Directive<[Spelling<"target teams distribute parallel for simd">]> {`。
- **L2616 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2616 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2617 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Aligned>,`.
  **L2617 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Aligned>,`。
- **L2618 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L2618 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L2619 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L2619 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L2620 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L2620 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L2621 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DefaultMap>,`.
  **L2621 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DefaultMap>,`。
- **L2622 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Depend>,`.
  **L2622 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Depend>,`。
- **L2623 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Device>,`.
  **L2623 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Device>,`。
- **L2624 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DistSchedule>,`.
  **L2624 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DistSchedule>,`。

### Lines 2625-2656

````tablegen
    VersionedClause<OMPC_DynGroupprivate, 61>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_HasDeviceAddr, 51>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_IsDevicePtr>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Linear>,
    VersionedClause<OMPC_Map>,
    VersionedClause<OMPC_Message, 60>,
    VersionedClause<OMPC_NonTemporal, 50>,
    VersionedClause<OMPC_NoWait>,
    VersionedClause<OMPC_NumTeams>,
    VersionedClause<OMPC_NumThreads>,
    VersionedClause<OMPC_OMPX_Attribute>,
    VersionedClause<OMPC_Order, 50>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_ProcBind>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_SafeLen>,
    VersionedClause<OMPC_Schedule>,
    VersionedClause<OMPC_Severity, 60>,
    VersionedClause<OMPC_Shared>,
    VersionedClause<OMPC_SimdLen>,
    VersionedClause<OMPC_ThreadLimit>,
    VersionedClause<OMPC_UsesAllocators, 50>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_OMPX_DynCGroupMem>,
  ];
  let leafConstructs =
      [OMP_Target, OMP_Teams, OMP_Distribute, OMP_Parallel, OMP_For, OMP_Simd];
  let category = CA_Executable;
````
- **L2625 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DynGroupprivate, 61>,`.
  **L2625 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DynGroupprivate, 61>,`。
- **L2626 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L2626 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L2627 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_HasDeviceAddr, 51>,`.
  **L2627 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_HasDeviceAddr, 51>,`。
- **L2628 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L2628 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L2629 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_IsDevicePtr>,`.
  **L2629 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_IsDevicePtr>,`。
- **L2630 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L2630 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L2631 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Linear>,`.
  **L2631 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Linear>,`。
- **L2632 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Map>,`.
  **L2632 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Map>,`。
- **L2633 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Message, 60>,`.
  **L2633 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Message, 60>,`。
- **L2634 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NonTemporal, 50>,`.
  **L2634 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NonTemporal, 50>,`。
- **L2635 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoWait>,`.
  **L2635 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoWait>,`。
- **L2636 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumTeams>,`.
  **L2636 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumTeams>,`。
- **L2637 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumThreads>,`.
  **L2637 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumThreads>,`。
- **L2638 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_Attribute>,`.
  **L2638 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_Attribute>,`。
- **L2639 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Order, 50>,`.
  **L2639 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Order, 50>,`。
- **L2640 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L2640 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L2641 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ProcBind>,`.
  **L2641 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ProcBind>,`。
- **L2642 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L2642 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L2643 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SafeLen>,`.
  **L2643 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SafeLen>,`。
- **L2644 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Schedule>,`.
  **L2644 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Schedule>,`。
- **L2645 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Severity, 60>,`.
  **L2645 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Severity, 60>,`。
- **L2646 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L2646 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L2647 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SimdLen>,`.
  **L2647 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SimdLen>,`。
- **L2648 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ThreadLimit>,`.
  **L2648 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ThreadLimit>,`。
- **L2649 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_UsesAllocators, 50>,`.
  **L2649 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_UsesAllocators, 50>,`。
- **L2650 EN**: Executes a standalone statement or declaration: `];`.
  **L2650 CN**: 执行一条独立语句或声明：`];`。
- **L2651 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2651 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2652 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_DynCGroupMem>,`.
  **L2652 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_DynCGroupMem>,`。
- **L2653 EN**: Executes a standalone statement or declaration: `];`.
  **L2653 CN**: 执行一条独立语句或声明：`];`。
- **L2654 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2654 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2655 EN**: Executes a standalone statement or declaration: `[OMP_Target, OMP_Teams, OMP_Distribute, OMP_Parallel, OMP_For, OMP_Simd];`.
  **L2655 CN**: 执行一条独立语句或声明：`[OMP_Target, OMP_Teams, OMP_Distribute, OMP_Parallel, OMP_For, OMP_Simd];`。
- **L2656 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2656 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 2657-2688

````tablegen
  let languages = [L_C];
}
def OMP_TargetTeamsDistributeSimd
    : Directive<[Spelling<"target teams distribute simd">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Aligned>,
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Depend>,
    VersionedClause<OMPC_DynGroupprivate, 61>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_HasDeviceAddr, 51>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_IsDevicePtr>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Linear>,
    VersionedClause<OMPC_Map>,
    VersionedClause<OMPC_NonTemporal, 50>,
    VersionedClause<OMPC_OMPX_Attribute>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_Shared>,
    VersionedClause<OMPC_UsesAllocators, 50>,
    VersionedClause<OMPC_Default, 60>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_DefaultMap>,
    VersionedClause<OMPC_Device>,
    VersionedClause<OMPC_DistSchedule>,
    VersionedClause<OMPC_NoWait>,
    VersionedClause<OMPC_NumTeams>,
    VersionedClause<OMPC_OMPX_DynCGroupMem>,
````
- **L2657 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2657 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2658 EN**: Closes the current lexical scope or compound statement.
  **L2658 CN**: 结束当前词法作用域或复合语句块。
- **L2659 EN**: Declares TableGen def `OMP_TargetTeamsDistributeSimd`.
  **L2659 CN**: 声明 TableGen def `OMP_TargetTeamsDistributeSimd`。
- **L2660 EN**: Continues the surrounding expression or declaration: `: Directive<[Spelling<"target teams distribute simd">]> {`.
  **L2660 CN**: 继续构造周围的表达式或声明：`: Directive<[Spelling<"target teams distribute simd">]> {`。
- **L2661 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2661 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2662 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Aligned>,`.
  **L2662 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Aligned>,`。
- **L2663 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L2663 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L2664 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Depend>,`.
  **L2664 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Depend>,`。
- **L2665 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DynGroupprivate, 61>,`.
  **L2665 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DynGroupprivate, 61>,`。
- **L2666 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L2666 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L2667 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_HasDeviceAddr, 51>,`.
  **L2667 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_HasDeviceAddr, 51>,`。
- **L2668 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L2668 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L2669 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_IsDevicePtr>,`.
  **L2669 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_IsDevicePtr>,`。
- **L2670 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L2670 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L2671 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Linear>,`.
  **L2671 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Linear>,`。
- **L2672 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Map>,`.
  **L2672 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Map>,`。
- **L2673 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NonTemporal, 50>,`.
  **L2673 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NonTemporal, 50>,`。
- **L2674 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_Attribute>,`.
  **L2674 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_Attribute>,`。
- **L2675 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L2675 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L2676 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L2676 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L2677 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L2677 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L2678 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_UsesAllocators, 50>,`.
  **L2678 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_UsesAllocators, 50>,`。
- **L2679 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default, 60>,`.
  **L2679 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default, 60>,`。
- **L2680 EN**: Executes a standalone statement or declaration: `];`.
  **L2680 CN**: 执行一条独立语句或声明：`];`。
- **L2681 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2681 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2682 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L2682 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L2683 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DefaultMap>,`.
  **L2683 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DefaultMap>,`。
- **L2684 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Device>,`.
  **L2684 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Device>,`。
- **L2685 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DistSchedule>,`.
  **L2685 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DistSchedule>,`。
- **L2686 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoWait>,`.
  **L2686 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoWait>,`。
- **L2687 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumTeams>,`.
  **L2687 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumTeams>,`。
- **L2688 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_DynCGroupMem>,`.
  **L2688 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_DynCGroupMem>,`。

### Lines 2689-2720

````tablegen
    VersionedClause<OMPC_Order, 50>,
    VersionedClause<OMPC_SafeLen>,
    VersionedClause<OMPC_SimdLen>,
    VersionedClause<OMPC_ThreadLimit>,
  ];
  let leafConstructs = [OMP_Target, OMP_Teams, OMP_Distribute, OMP_Simd];
  let category = CA_Executable;
}
def OMP_TargetTeamsWorkdistribute : Directive<[Spelling<"target teams workdistribute">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Depend>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_HasDeviceAddr, 51>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_IsDevicePtr>,
    VersionedClause<OMPC_Map>,
    VersionedClause<OMPC_OMPX_Attribute>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_Shared>,
    VersionedClause<OMPC_UsesAllocators, 50>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_DefaultMap>,
    VersionedClause<OMPC_Device>,
    VersionedClause<OMPC_NoWait>,
    VersionedClause<OMPC_NumTeams>,
    VersionedClause<OMPC_OMPX_DynCGroupMem>,
    VersionedClause<OMPC_OMPX_Bare>,
    VersionedClause<OMPC_ThreadLimit>,
````
- **L2689 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Order, 50>,`.
  **L2689 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Order, 50>,`。
- **L2690 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SafeLen>,`.
  **L2690 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SafeLen>,`。
- **L2691 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SimdLen>,`.
  **L2691 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SimdLen>,`。
- **L2692 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ThreadLimit>,`.
  **L2692 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ThreadLimit>,`。
- **L2693 EN**: Executes a standalone statement or declaration: `];`.
  **L2693 CN**: 执行一条独立语句或声明：`];`。
- **L2694 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2694 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2695 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2695 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2696 EN**: Closes the current lexical scope or compound statement.
  **L2696 CN**: 结束当前词法作用域或复合语句块。
- **L2697 EN**: Declares TableGen def `OMP_TargetTeamsWorkdistribute`.
  **L2697 CN**: 声明 TableGen def `OMP_TargetTeamsWorkdistribute`。
- **L2698 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2698 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2699 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L2699 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L2700 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Depend>,`.
  **L2700 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Depend>,`。
- **L2701 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L2701 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L2702 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_HasDeviceAddr, 51>,`.
  **L2702 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_HasDeviceAddr, 51>,`。
- **L2703 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L2703 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L2704 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_IsDevicePtr>,`.
  **L2704 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_IsDevicePtr>,`。
- **L2705 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Map>,`.
  **L2705 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Map>,`。
- **L2706 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_Attribute>,`.
  **L2706 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_Attribute>,`。
- **L2707 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L2707 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L2708 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L2708 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L2709 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L2709 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L2710 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_UsesAllocators, 50>,`.
  **L2710 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_UsesAllocators, 50>,`。
- **L2711 EN**: Executes a standalone statement or declaration: `];`.
  **L2711 CN**: 执行一条独立语句或声明：`];`。
- **L2712 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2712 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2713 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L2713 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L2714 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DefaultMap>,`.
  **L2714 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DefaultMap>,`。
- **L2715 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Device>,`.
  **L2715 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Device>,`。
- **L2716 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoWait>,`.
  **L2716 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoWait>,`。
- **L2717 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumTeams>,`.
  **L2717 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumTeams>,`。
- **L2718 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_DynCGroupMem>,`.
  **L2718 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_DynCGroupMem>,`。
- **L2719 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_Bare>,`.
  **L2719 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_Bare>,`。
- **L2720 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ThreadLimit>,`.
  **L2720 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ThreadLimit>,`。

### Lines 2721-2752

````tablegen
  ];
  let leafConstructs = [OMP_Target, OMP_Teams, OMP_Workdistribute];
  let category = CA_Executable;
  let languages = [L_Fortran];
}
def OMP_target_teams_loop : Directive<[Spelling<"target teams loop">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_DefaultMap>,
    VersionedClause<OMPC_Depend>,
    VersionedClause<OMPC_Device>,
    VersionedClause<OMPC_DynGroupprivate, 61>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_HasDeviceAddr, 51>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_IsDevicePtr>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Map>,
    VersionedClause<OMPC_OMPX_Attribute>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_Shared>,
    VersionedClause<OMPC_UsesAllocators, 50>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_Bind, 50>,
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_NoWait>,
    VersionedClause<OMPC_NumTeams>,
    VersionedClause<OMPC_OMPX_DynCGroupMem>,
    VersionedClause<OMPC_Order>,
````
- **L2721 EN**: Executes a standalone statement or declaration: `];`.
  **L2721 CN**: 执行一条独立语句或声明：`];`。
- **L2722 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2722 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2723 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2723 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2724 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2724 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2725 EN**: Closes the current lexical scope or compound statement.
  **L2725 CN**: 结束当前词法作用域或复合语句块。
- **L2726 EN**: Declares TableGen def `OMP_target_teams_loop`.
  **L2726 CN**: 声明 TableGen def `OMP_target_teams_loop`。
- **L2727 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2727 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2728 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L2728 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L2729 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DefaultMap>,`.
  **L2729 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DefaultMap>,`。
- **L2730 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Depend>,`.
  **L2730 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Depend>,`。
- **L2731 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Device>,`.
  **L2731 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Device>,`。
- **L2732 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DynGroupprivate, 61>,`.
  **L2732 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DynGroupprivate, 61>,`。
- **L2733 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L2733 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L2734 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_HasDeviceAddr, 51>,`.
  **L2734 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_HasDeviceAddr, 51>,`。
- **L2735 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L2735 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L2736 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_IsDevicePtr>,`.
  **L2736 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_IsDevicePtr>,`。
- **L2737 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L2737 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L2738 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Map>,`.
  **L2738 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Map>,`。
- **L2739 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_Attribute>,`.
  **L2739 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_Attribute>,`。
- **L2740 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L2740 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L2741 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L2741 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L2742 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L2742 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L2743 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_UsesAllocators, 50>,`.
  **L2743 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_UsesAllocators, 50>,`。
- **L2744 EN**: Executes a standalone statement or declaration: `];`.
  **L2744 CN**: 执行一条独立语句或声明：`];`。
- **L2745 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2745 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2746 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Bind, 50>,`.
  **L2746 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Bind, 50>,`。
- **L2747 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L2747 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L2748 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L2748 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L2749 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoWait>,`.
  **L2749 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoWait>,`。
- **L2750 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumTeams>,`.
  **L2750 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumTeams>,`。
- **L2751 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_DynCGroupMem>,`.
  **L2751 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_DynCGroupMem>,`。
- **L2752 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Order>,`.
  **L2752 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Order>,`。

### Lines 2753-2784

````tablegen
    VersionedClause<OMPC_ThreadLimit>,
  ];
  let leafConstructs = [OMP_Target, OMP_Teams, OMP_loop];
  let category = CA_Executable;
}
def OMP_TaskLoopSimd : Directive<[Spelling<"taskloop simd">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Aligned>,
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_InReduction>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Linear>,
    VersionedClause<OMPC_Mergeable>,
    VersionedClause<OMPC_NoGroup>,
    VersionedClause<OMPC_NonTemporal, 50>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_Shared>,
    VersionedClause<OMPC_Untied>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_Final>,
    VersionedClause<OMPC_Order, 50>,
    VersionedClause<OMPC_Priority>,
    VersionedClause<OMPC_SafeLen>,
    VersionedClause<OMPC_SimdLen>,
  ];
  let allowedExclusiveClauses = [
````
- **L2753 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ThreadLimit>,`.
  **L2753 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ThreadLimit>,`。
- **L2754 EN**: Executes a standalone statement or declaration: `];`.
  **L2754 CN**: 执行一条独立语句或声明：`];`。
- **L2755 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2755 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2756 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2756 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2757 EN**: Closes the current lexical scope or compound statement.
  **L2757 CN**: 结束当前词法作用域或复合语句块。
- **L2758 EN**: Declares TableGen def `OMP_TaskLoopSimd`.
  **L2758 CN**: 声明 TableGen def `OMP_TaskLoopSimd`。
- **L2759 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2759 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2760 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Aligned>,`.
  **L2760 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Aligned>,`。
- **L2761 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L2761 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L2762 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L2762 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L2763 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L2763 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L2764 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L2764 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L2765 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_InReduction>,`.
  **L2765 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_InReduction>,`。
- **L2766 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L2766 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L2767 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Linear>,`.
  **L2767 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Linear>,`。
- **L2768 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Mergeable>,`.
  **L2768 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Mergeable>,`。
- **L2769 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NoGroup>,`.
  **L2769 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NoGroup>,`。
- **L2770 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NonTemporal, 50>,`.
  **L2770 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NonTemporal, 50>,`。
- **L2771 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L2771 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L2772 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L2772 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L2773 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L2773 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L2774 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Untied>,`.
  **L2774 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Untied>,`。
- **L2775 EN**: Executes a standalone statement or declaration: `];`.
  **L2775 CN**: 执行一条独立语句或声明：`];`。
- **L2776 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2776 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2777 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L2777 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L2778 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Final>,`.
  **L2778 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Final>,`。
- **L2779 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Order, 50>,`.
  **L2779 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Order, 50>,`。
- **L2780 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Priority>,`.
  **L2780 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Priority>,`。
- **L2781 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SafeLen>,`.
  **L2781 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SafeLen>,`。
- **L2782 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SimdLen>,`.
  **L2782 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SimdLen>,`。
- **L2783 EN**: Executes a standalone statement or declaration: `];`.
  **L2783 CN**: 执行一条独立语句或声明：`];`。
- **L2784 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2784 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 2785-2816

````tablegen
    VersionedClause<OMPC_GrainSize>,
    VersionedClause<OMPC_NumTasks>,
  ];
  let leafConstructs = [OMP_TaskLoop, OMP_Simd];
  let category = CA_Executable;
}
def OMP_TeamsDistribute : Directive<[Spelling<"teams distribute">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_DistSchedule>,
    VersionedClause<OMPC_DynGroupprivate, 61>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_NumTeams>,
    VersionedClause<OMPC_OMPX_Attribute>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_Shared>,
    VersionedClause<OMPC_ThreadLimit>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_Order, 50>,
  ];
  let leafConstructs = [OMP_Teams, OMP_Distribute];
  let category = CA_Executable;
}
def OMP_TeamsDistributeParallelDo
    : Directive<[Spelling<"teams distribute parallel do">]> {
  let allowedClauses = [
````
- **L2785 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_GrainSize>,`.
  **L2785 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_GrainSize>,`。
- **L2786 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumTasks>,`.
  **L2786 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumTasks>,`。
- **L2787 EN**: Executes a standalone statement or declaration: `];`.
  **L2787 CN**: 执行一条独立语句或声明：`];`。
- **L2788 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2788 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2789 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2789 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2790 EN**: Closes the current lexical scope or compound statement.
  **L2790 CN**: 结束当前词法作用域或复合语句块。
- **L2791 EN**: Declares TableGen def `OMP_TeamsDistribute`.
  **L2791 CN**: 声明 TableGen def `OMP_TeamsDistribute`。
- **L2792 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2792 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2793 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L2793 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L2794 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L2794 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L2795 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L2795 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L2796 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DistSchedule>,`.
  **L2796 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DistSchedule>,`。
- **L2797 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DynGroupprivate, 61>,`.
  **L2797 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DynGroupprivate, 61>,`。
- **L2798 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L2798 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L2799 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L2799 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L2800 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumTeams>,`.
  **L2800 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumTeams>,`。
- **L2801 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_Attribute>,`.
  **L2801 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_Attribute>,`。
- **L2802 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L2802 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L2803 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L2803 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L2804 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L2804 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L2805 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ThreadLimit>,`.
  **L2805 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ThreadLimit>,`。
- **L2806 EN**: Executes a standalone statement or declaration: `];`.
  **L2806 CN**: 执行一条独立语句或声明：`];`。
- **L2807 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2807 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2808 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L2808 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L2809 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Order, 50>,`.
  **L2809 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Order, 50>,`。
- **L2810 EN**: Executes a standalone statement or declaration: `];`.
  **L2810 CN**: 执行一条独立语句或声明：`];`。
- **L2811 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2811 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2812 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2812 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2813 EN**: Closes the current lexical scope or compound statement.
  **L2813 CN**: 结束当前词法作用域或复合语句块。
- **L2814 EN**: Declares TableGen def `OMP_TeamsDistributeParallelDo`.
  **L2814 CN**: 声明 TableGen def `OMP_TeamsDistributeParallelDo`。
- **L2815 EN**: Continues the surrounding expression or declaration: `: Directive<[Spelling<"teams distribute parallel do">]> {`.
  **L2815 CN**: 继续构造周围的表达式或声明：`: Directive<[Spelling<"teams distribute parallel do">]> {`。
- **L2816 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2816 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 2817-2848

````tablegen
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Copyin>,
    VersionedClause<OMPC_DynGroupprivate, 61>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Linear>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_Shared>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_DistSchedule>,
    VersionedClause<OMPC_Message, 60>,
    VersionedClause<OMPC_NumTeams>,
    VersionedClause<OMPC_NumThreads>,
    VersionedClause<OMPC_Order, 50>,
    VersionedClause<OMPC_ProcBind>,
    VersionedClause<OMPC_Schedule>,
    VersionedClause<OMPC_Severity, 60>,
    VersionedClause<OMPC_ThreadLimit>,
  ];
  let leafConstructs = [OMP_Teams, OMP_Distribute, OMP_Parallel, OMP_Do];
  let category = CA_Executable;
  let languages = [L_Fortran];
}
def OMP_TeamsDistributeParallelDoSimd
    : Directive<[Spelling<"teams distribute parallel do simd">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Aligned>,
````
- **L2817 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L2817 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L2818 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Copyin>,`.
  **L2818 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Copyin>,`。
- **L2819 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DynGroupprivate, 61>,`.
  **L2819 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DynGroupprivate, 61>,`。
- **L2820 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L2820 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L2821 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L2821 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L2822 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L2822 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L2823 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Linear>,`.
  **L2823 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Linear>,`。
- **L2824 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L2824 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L2825 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L2825 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L2826 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L2826 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L2827 EN**: Executes a standalone statement or declaration: `];`.
  **L2827 CN**: 执行一条独立语句或声明：`];`。
- **L2828 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2828 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2829 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L2829 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L2830 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L2830 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L2831 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DistSchedule>,`.
  **L2831 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DistSchedule>,`。
- **L2832 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Message, 60>,`.
  **L2832 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Message, 60>,`。
- **L2833 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumTeams>,`.
  **L2833 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumTeams>,`。
- **L2834 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumThreads>,`.
  **L2834 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumThreads>,`。
- **L2835 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Order, 50>,`.
  **L2835 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Order, 50>,`。
- **L2836 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ProcBind>,`.
  **L2836 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ProcBind>,`。
- **L2837 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Schedule>,`.
  **L2837 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Schedule>,`。
- **L2838 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Severity, 60>,`.
  **L2838 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Severity, 60>,`。
- **L2839 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ThreadLimit>,`.
  **L2839 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ThreadLimit>,`。
- **L2840 EN**: Executes a standalone statement or declaration: `];`.
  **L2840 CN**: 执行一条独立语句或声明：`];`。
- **L2841 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2841 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2842 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2842 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2843 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2843 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2844 EN**: Closes the current lexical scope or compound statement.
  **L2844 CN**: 结束当前词法作用域或复合语句块。
- **L2845 EN**: Declares TableGen def `OMP_TeamsDistributeParallelDoSimd`.
  **L2845 CN**: 声明 TableGen def `OMP_TeamsDistributeParallelDoSimd`。
- **L2846 EN**: Continues the surrounding expression or declaration: `: Directive<[Spelling<"teams distribute parallel do simd">]> {`.
  **L2846 CN**: 继续构造周围的表达式或声明：`: Directive<[Spelling<"teams distribute parallel do simd">]> {`。
- **L2847 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2847 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2848 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Aligned>,`.
  **L2848 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Aligned>,`。

### Lines 2849-2880

````tablegen
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_DynGroupprivate, 61>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Linear>,
    VersionedClause<OMPC_NonTemporal>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_Shared>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_DistSchedule>,
    VersionedClause<OMPC_Message, 60>,
    VersionedClause<OMPC_NumTeams>,
    VersionedClause<OMPC_NumThreads>,
    VersionedClause<OMPC_Order, 50>,
    VersionedClause<OMPC_ProcBind>,
    VersionedClause<OMPC_SafeLen>,
    VersionedClause<OMPC_Schedule>,
    VersionedClause<OMPC_Severity, 60>,
    VersionedClause<OMPC_SimdLen>,
    VersionedClause<OMPC_ThreadLimit>,
  ];
  let leafConstructs =
      [OMP_Teams, OMP_Distribute, OMP_Parallel, OMP_Do, OMP_Simd];
  let category = CA_Executable;
  let languages = [L_Fortran];
}
def OMP_TeamsDistributeParallelFor
````
- **L2849 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L2849 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L2850 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DynGroupprivate, 61>,`.
  **L2850 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DynGroupprivate, 61>,`。
- **L2851 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L2851 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L2852 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L2852 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L2853 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L2853 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L2854 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Linear>,`.
  **L2854 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Linear>,`。
- **L2855 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NonTemporal>,`.
  **L2855 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NonTemporal>,`。
- **L2856 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L2856 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L2857 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L2857 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L2858 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L2858 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L2859 EN**: Executes a standalone statement or declaration: `];`.
  **L2859 CN**: 执行一条独立语句或声明：`];`。
- **L2860 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2860 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2861 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L2861 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L2862 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L2862 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L2863 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DistSchedule>,`.
  **L2863 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DistSchedule>,`。
- **L2864 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Message, 60>,`.
  **L2864 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Message, 60>,`。
- **L2865 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumTeams>,`.
  **L2865 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumTeams>,`。
- **L2866 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumThreads>,`.
  **L2866 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumThreads>,`。
- **L2867 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Order, 50>,`.
  **L2867 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Order, 50>,`。
- **L2868 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ProcBind>,`.
  **L2868 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ProcBind>,`。
- **L2869 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SafeLen>,`.
  **L2869 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SafeLen>,`。
- **L2870 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Schedule>,`.
  **L2870 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Schedule>,`。
- **L2871 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Severity, 60>,`.
  **L2871 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Severity, 60>,`。
- **L2872 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SimdLen>,`.
  **L2872 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SimdLen>,`。
- **L2873 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ThreadLimit>,`.
  **L2873 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ThreadLimit>,`。
- **L2874 EN**: Executes a standalone statement or declaration: `];`.
  **L2874 CN**: 执行一条独立语句或声明：`];`。
- **L2875 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2875 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2876 EN**: Executes a standalone statement or declaration: `[OMP_Teams, OMP_Distribute, OMP_Parallel, OMP_Do, OMP_Simd];`.
  **L2876 CN**: 执行一条独立语句或声明：`[OMP_Teams, OMP_Distribute, OMP_Parallel, OMP_Do, OMP_Simd];`。
- **L2877 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2877 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2878 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2878 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2879 EN**: Closes the current lexical scope or compound statement.
  **L2879 CN**: 结束当前词法作用域或复合语句块。
- **L2880 EN**: Declares TableGen def `OMP_TeamsDistributeParallelFor`.
  **L2880 CN**: 声明 TableGen def `OMP_TeamsDistributeParallelFor`。

### Lines 2881-2912

````tablegen
    : Directive<[Spelling<"teams distribute parallel for">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_Copyin>,
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_DistSchedule>,
    VersionedClause<OMPC_DynGroupprivate, 61>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Message, 60>,
    VersionedClause<OMPC_NumTeams>,
    VersionedClause<OMPC_NumThreads>,
    VersionedClause<OMPC_OMPX_Attribute>,
    VersionedClause<OMPC_Order, 50>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_ProcBind>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_Schedule>,
    VersionedClause<OMPC_Severity, 60>,
    VersionedClause<OMPC_Shared>,
    VersionedClause<OMPC_ThreadLimit>,
  ];
  let leafConstructs = [OMP_Teams, OMP_Distribute, OMP_Parallel, OMP_For];
  let category = CA_Executable;
  let languages = [L_C];
}
def OMP_TeamsDistributeParallelForSimd
    : Directive<[Spelling<"teams distribute parallel for simd">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Aligned>,
````
- **L2881 EN**: Continues the surrounding expression or declaration: `: Directive<[Spelling<"teams distribute parallel for">]> {`.
  **L2881 CN**: 继续构造周围的表达式或声明：`: Directive<[Spelling<"teams distribute parallel for">]> {`。
- **L2882 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2882 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2883 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L2883 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L2884 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L2884 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L2885 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Copyin>,`.
  **L2885 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Copyin>,`。
- **L2886 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L2886 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L2887 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DistSchedule>,`.
  **L2887 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DistSchedule>,`。
- **L2888 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DynGroupprivate, 61>,`.
  **L2888 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DynGroupprivate, 61>,`。
- **L2889 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L2889 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L2890 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L2890 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L2891 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L2891 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L2892 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Message, 60>,`.
  **L2892 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Message, 60>,`。
- **L2893 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumTeams>,`.
  **L2893 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumTeams>,`。
- **L2894 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumThreads>,`.
  **L2894 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumThreads>,`。
- **L2895 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_Attribute>,`.
  **L2895 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_Attribute>,`。
- **L2896 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Order, 50>,`.
  **L2896 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Order, 50>,`。
- **L2897 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L2897 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L2898 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ProcBind>,`.
  **L2898 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ProcBind>,`。
- **L2899 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L2899 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L2900 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Schedule>,`.
  **L2900 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Schedule>,`。
- **L2901 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Severity, 60>,`.
  **L2901 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Severity, 60>,`。
- **L2902 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L2902 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L2903 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ThreadLimit>,`.
  **L2903 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ThreadLimit>,`。
- **L2904 EN**: Executes a standalone statement or declaration: `];`.
  **L2904 CN**: 执行一条独立语句或声明：`];`。
- **L2905 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2905 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2906 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2906 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2907 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2907 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2908 EN**: Closes the current lexical scope or compound statement.
  **L2908 CN**: 结束当前词法作用域或复合语句块。
- **L2909 EN**: Declares TableGen def `OMP_TeamsDistributeParallelForSimd`.
  **L2909 CN**: 声明 TableGen def `OMP_TeamsDistributeParallelForSimd`。
- **L2910 EN**: Continues the surrounding expression or declaration: `: Directive<[Spelling<"teams distribute parallel for simd">]> {`.
  **L2910 CN**: 继续构造周围的表达式或声明：`: Directive<[Spelling<"teams distribute parallel for simd">]> {`。
- **L2911 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2911 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2912 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Aligned>,`.
  **L2912 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Aligned>,`。

### Lines 2913-2944

````tablegen
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_DistSchedule>,
    VersionedClause<OMPC_DynGroupprivate, 61>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_If>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Linear>,
    VersionedClause<OMPC_Message, 60>,
    VersionedClause<OMPC_NonTemporal, 50>,
    VersionedClause<OMPC_NumTeams>,
    VersionedClause<OMPC_NumThreads>,
    VersionedClause<OMPC_OMPX_Attribute>,
    VersionedClause<OMPC_Order, 50>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_ProcBind>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_SafeLen>,
    VersionedClause<OMPC_Schedule>,
    VersionedClause<OMPC_Severity, 60>,
    VersionedClause<OMPC_Shared>,
    VersionedClause<OMPC_SimdLen>,
    VersionedClause<OMPC_ThreadLimit>,
  ];
  let leafConstructs =
      [OMP_Teams, OMP_Distribute, OMP_Parallel, OMP_For, OMP_Simd];
  let category = CA_Executable;
  let languages = [L_C];
}
def OMP_TeamsDistributeSimd : Directive<[Spelling<"teams distribute simd">]> {
  let allowedClauses = [
````
- **L2913 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L2913 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L2914 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L2914 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L2915 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L2915 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L2916 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DistSchedule>,`.
  **L2916 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DistSchedule>,`。
- **L2917 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DynGroupprivate, 61>,`.
  **L2917 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DynGroupprivate, 61>,`。
- **L2918 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L2918 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L2919 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If>,`.
  **L2919 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If>,`。
- **L2920 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L2920 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L2921 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Linear>,`.
  **L2921 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Linear>,`。
- **L2922 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Message, 60>,`.
  **L2922 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Message, 60>,`。
- **L2923 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NonTemporal, 50>,`.
  **L2923 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NonTemporal, 50>,`。
- **L2924 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumTeams>,`.
  **L2924 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumTeams>,`。
- **L2925 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumThreads>,`.
  **L2925 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumThreads>,`。
- **L2926 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_Attribute>,`.
  **L2926 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_Attribute>,`。
- **L2927 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Order, 50>,`.
  **L2927 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Order, 50>,`。
- **L2928 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L2928 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L2929 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ProcBind>,`.
  **L2929 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ProcBind>,`。
- **L2930 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L2930 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L2931 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SafeLen>,`.
  **L2931 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SafeLen>,`。
- **L2932 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Schedule>,`.
  **L2932 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Schedule>,`。
- **L2933 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Severity, 60>,`.
  **L2933 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Severity, 60>,`。
- **L2934 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L2934 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L2935 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SimdLen>,`.
  **L2935 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SimdLen>,`。
- **L2936 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ThreadLimit>,`.
  **L2936 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ThreadLimit>,`。
- **L2937 EN**: Executes a standalone statement or declaration: `];`.
  **L2937 CN**: 执行一条独立语句或声明：`];`。
- **L2938 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2938 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2939 EN**: Executes a standalone statement or declaration: `[OMP_Teams, OMP_Distribute, OMP_Parallel, OMP_For, OMP_Simd];`.
  **L2939 CN**: 执行一条独立语句或声明：`[OMP_Teams, OMP_Distribute, OMP_Parallel, OMP_For, OMP_Simd];`。
- **L2940 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2940 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2941 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2941 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2942 EN**: Closes the current lexical scope or compound statement.
  **L2942 CN**: 结束当前词法作用域或复合语句块。
- **L2943 EN**: Declares TableGen def `OMP_TeamsDistributeSimd`.
  **L2943 CN**: 声明 TableGen def `OMP_TeamsDistributeSimd`。
- **L2944 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2944 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 2945-2976

````tablegen
    VersionedClause<OMPC_Aligned>,
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_DynGroupprivate, 61>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_If, 50>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_Linear>,
    VersionedClause<OMPC_NonTemporal, 50>,
    VersionedClause<OMPC_OMPX_Attribute>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_Shared>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_DistSchedule>,
    VersionedClause<OMPC_NumTeams>,
    VersionedClause<OMPC_Order, 50>,
    VersionedClause<OMPC_SafeLen>,
    VersionedClause<OMPC_SimdLen>,
    VersionedClause<OMPC_ThreadLimit>,
  ];
  let leafConstructs = [OMP_Teams, OMP_Distribute, OMP_Simd];
  let category = CA_Executable;
}
def OMP_TeamsWorkdistribute : Directive<[Spelling<"teams workdistribute">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_OMPX_Attribute>,
    VersionedClause<OMPC_Private>,
````
- **L2945 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Aligned>,`.
  **L2945 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Aligned>,`。
- **L2946 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L2946 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L2947 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DynGroupprivate, 61>,`.
  **L2947 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DynGroupprivate, 61>,`。
- **L2948 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L2948 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L2949 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If, 50>,`.
  **L2949 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If, 50>,`。
- **L2950 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L2950 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L2951 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Linear>,`.
  **L2951 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Linear>,`。
- **L2952 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NonTemporal, 50>,`.
  **L2952 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NonTemporal, 50>,`。
- **L2953 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_Attribute>,`.
  **L2953 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_Attribute>,`。
- **L2954 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L2954 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L2955 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L2955 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L2956 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L2956 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L2957 EN**: Executes a standalone statement or declaration: `];`.
  **L2957 CN**: 执行一条独立语句或声明：`];`。
- **L2958 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2958 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2959 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L2959 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L2960 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L2960 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L2961 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DistSchedule>,`.
  **L2961 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DistSchedule>,`。
- **L2962 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumTeams>,`.
  **L2962 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumTeams>,`。
- **L2963 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Order, 50>,`.
  **L2963 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Order, 50>,`。
- **L2964 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SafeLen>,`.
  **L2964 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SafeLen>,`。
- **L2965 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_SimdLen>,`.
  **L2965 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_SimdLen>,`。
- **L2966 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ThreadLimit>,`.
  **L2966 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ThreadLimit>,`。
- **L2967 EN**: Executes a standalone statement or declaration: `];`.
  **L2967 CN**: 执行一条独立语句或声明：`];`。
- **L2968 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2968 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2969 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2969 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2970 EN**: Closes the current lexical scope or compound statement.
  **L2970 CN**: 结束当前词法作用域或复合语句块。
- **L2971 EN**: Declares TableGen def `OMP_TeamsWorkdistribute`.
  **L2971 CN**: 声明 TableGen def `OMP_TeamsWorkdistribute`。
- **L2972 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2972 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2973 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L2973 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L2974 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L2974 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L2975 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_Attribute>,`.
  **L2975 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_Attribute>,`。
- **L2976 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L2976 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。

### Lines 2977-3008

````tablegen
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_Shared>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_If, 52>,
    VersionedClause<OMPC_NumTeams>,
    VersionedClause<OMPC_ThreadLimit>,
  ];
  let leafConstructs = [OMP_Teams, OMP_Workdistribute];
  let category = CA_Executable;
  let languages = [L_Fortran];
}
def OMP_teams_loop : Directive<[Spelling<"teams loop">]> {
  let allowedClauses = [
    VersionedClause<OMPC_Allocate>,
    VersionedClause<OMPC_DynGroupprivate, 61>,
    VersionedClause<OMPC_FirstPrivate>,
    VersionedClause<OMPC_LastPrivate>,
    VersionedClause<OMPC_OMPX_Attribute>,
    VersionedClause<OMPC_Private>,
    VersionedClause<OMPC_Reduction>,
    VersionedClause<OMPC_Shared>,
  ];
  let allowedOnceClauses = [
    VersionedClause<OMPC_Bind, 50>,
    VersionedClause<OMPC_Collapse>,
    VersionedClause<OMPC_Default>,
    VersionedClause<OMPC_NumTeams>,
    VersionedClause<OMPC_Order>,
    VersionedClause<OMPC_ThreadLimit>,
  ];
````
- **L2977 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L2977 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L2978 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L2978 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L2979 EN**: Executes a standalone statement or declaration: `];`.
  **L2979 CN**: 执行一条独立语句或声明：`];`。
- **L2980 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2980 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2981 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L2981 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L2982 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_If, 52>,`.
  **L2982 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_If, 52>,`。
- **L2983 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumTeams>,`.
  **L2983 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumTeams>,`。
- **L2984 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ThreadLimit>,`.
  **L2984 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ThreadLimit>,`。
- **L2985 EN**: Executes a standalone statement or declaration: `];`.
  **L2985 CN**: 执行一条独立语句或声明：`];`。
- **L2986 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2986 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2987 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2987 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2988 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2988 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2989 EN**: Closes the current lexical scope or compound statement.
  **L2989 CN**: 结束当前词法作用域或复合语句块。
- **L2990 EN**: Declares TableGen def `OMP_teams_loop`.
  **L2990 CN**: 声明 TableGen def `OMP_teams_loop`。
- **L2991 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2991 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2992 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Allocate>,`.
  **L2992 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Allocate>,`。
- **L2993 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_DynGroupprivate, 61>,`.
  **L2993 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_DynGroupprivate, 61>,`。
- **L2994 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_FirstPrivate>,`.
  **L2994 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_FirstPrivate>,`。
- **L2995 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_LastPrivate>,`.
  **L2995 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_LastPrivate>,`。
- **L2996 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_OMPX_Attribute>,`.
  **L2996 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_OMPX_Attribute>,`。
- **L2997 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Private>,`.
  **L2997 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Private>,`。
- **L2998 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Reduction>,`.
  **L2998 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Reduction>,`。
- **L2999 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Shared>,`.
  **L2999 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Shared>,`。
- **L3000 EN**: Executes a standalone statement or declaration: `];`.
  **L3000 CN**: 执行一条独立语句或声明：`];`。
- **L3001 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L3001 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3002 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Bind, 50>,`.
  **L3002 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Bind, 50>,`。
- **L3003 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Collapse>,`.
  **L3003 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Collapse>,`。
- **L3004 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Default>,`.
  **L3004 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Default>,`。
- **L3005 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_NumTeams>,`.
  **L3005 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_NumTeams>,`。
- **L3006 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_Order>,`.
  **L3006 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_Order>,`。
- **L3007 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VersionedClause<OMPC_ThreadLimit>,`.
  **L3007 CN**: 继续一个多行参数列表、初始化器或聚合项：`VersionedClause<OMPC_ThreadLimit>,`。
- **L3008 EN**: Executes a standalone statement or declaration: `];`.
  **L3008 CN**: 执行一条独立语句或声明：`];`。

### Lines 3009-3011

````tablegen
  let leafConstructs = [OMP_Teams, OMP_loop];
  let category = CA_Executable;
}
````
- **L3009 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L3009 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3010 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L3010 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3011 EN**: Closes the current lexical scope or compound statement.
  **L3011 CN**: 结束当前词法作用域或复合语句块。

## Key Concepts / 关键概念

- **TableGen record modeling / TableGen 记录建模**
- **ORC JIT runtime and remote execution / ORC JIT 运行时与远程执行**
- **OpenMP IR construction / OpenMP IR 构建**
- **Typed error propagation / 类型化错误传播**
- **Parallel runtime integration / 并行运行时集成**

## Dependencies / 依赖关系

- `llvm/Frontend/Directive/DirectiveBase.td`: Provides frontend-facing integration helpers. / 提供面向前端的集成辅助组件。
