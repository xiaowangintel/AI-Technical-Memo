# DiagnosticParseKinds.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/include/clang/Basic/DiagnosticParseKinds.td`
- **Repository / 仓库**: `/root/xw/llvm-project`
- **Purpose (EN)**: Declares foundational compiler metadata, diagnostics, target descriptions, or builtin-related definitions for `DiagnosticParseKinds`.
- **Purpose (CN)**: 声明与 `DiagnosticParseKinds` 相关的编译器基础元数据、诊断信息、目标描述或 builtin 定义。
- **Line Count / 行数**: 1898

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````tablegen
//==--- DiagnosticParseKinds.td - libparse diagnostics --------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
// Parser Diagnostics
//===----------------------------------------------------------------------===//

let Component = "Parse" in {
let CategoryName = "Parse Issue" in {
// C++11 compatibility with C++98.
defm enum_fixed_underlying_type : CXX11Compat<
  "enumeration types with a fixed underlying type are",
  /*ext_warn=*/false>;
}

def err_asm_qualifier_ignored : Error<
  "expected 'volatile', 'inline', 'goto', or '('">, CatInlineAsm;
def err_global_asm_qualifier_ignored : Error<
  "meaningless '%0' on asm outside function">, CatInlineAsm;
````
- **L1 EN**: Comment explains nearby logic, constraints, or intent: `DiagnosticParseKinds.td - libparse diagnostics`.
  **L1 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DiagnosticParseKinds.td - libparse diagnostics`。
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
- **L8 EN**: Blank line separating nearby declarations or logic blocks.
  **L8 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L9 EN**: Banner comment marking a file or section boundary.
  **L9 CN**: 横幅注释，用于标记文件或章节边界。
- **L10 EN**: Comment explains nearby logic, constraints, or intent: `Parser Diagnostics`.
  **L10 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Parser Diagnostics`。
- **L11 EN**: Banner comment marking a file or section boundary.
  **L11 CN**: 横幅注释，用于标记文件或章节边界。
- **L12 EN**: Blank line separating nearby declarations or logic blocks.
  **L12 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L13 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Component = "Parse" in {`.
  **L13 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Component = "Parse" in {`。
- **L14 EN**: Assigns a TableGen property that affects following records or inherited fields: `let CategoryName = "Parse Issue" in {`.
  **L14 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let CategoryName = "Parse Issue" in {`。
- **L15 EN**: Comment explains nearby logic, constraints, or intent: `C++11 compatibility with C++98.`.
  **L15 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C++11 compatibility with C++98.`。
- **L16 EN**: Declares TableGen defm record `enum_fixed_underlying_type`.
  **L16 CN**: 声明 TableGen defm 记录 `enum_fixed_underlying_type`。
- **L17 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"enumeration types with a fixed underlying type are",`.
  **L17 CN**: 继续一个多行参数列表、初始化器或聚合项：`"enumeration types with a fixed underlying type are",`。
- **L18 EN**: Comment explains nearby logic, constraints, or intent: `ext_warn false>;`.
  **L18 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ext_warn false>;`。
- **L19 EN**: Closes the current lexical scope or compound statement.
  **L19 CN**: 结束当前词法作用域或复合语句块。
- **L20 EN**: Blank line separating nearby declarations or logic blocks.
  **L20 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L21 EN**: Declares TableGen def record `err_asm_qualifier_ignored`.
  **L21 CN**: 声明 TableGen def 记录 `err_asm_qualifier_ignored`。
- **L22 EN**: Executes a call or declaration centered on `'`.
  **L22 CN**: 执行以 `'` 为核心的调用或声明。
- **L23 EN**: Declares TableGen def record `err_global_asm_qualifier_ignored`.
  **L23 CN**: 声明 TableGen def 记录 `err_global_asm_qualifier_ignored`。
- **L24 EN**: Adds a standalone statement or declaration: `"meaningless '%0' on asm outside function">, CatInlineAsm;`.
  **L24 CN**: 添加一条独立语句或声明：`"meaningless '%0' on asm outside function">, CatInlineAsm;`。

### Lines 25-48

````tablegen

let CategoryName = "Inline Assembly Issue" in {
def err_asm_empty : Error<"__asm used with no assembly instructions">;
def err_inline_ms_asm_parsing : Error<"%0">;
def err_msasm_unsupported_arch : Error<
  "unsupported architecture '%0' for MS-style inline assembly">;
def err_msasm_unable_to_create_target : Error<
  "MS-style inline assembly is not available: %0">;
def err_gnu_inline_asm_disabled : Error<
  "GNU-style inline assembly is disabled">;
def err_asm_duplicate_qual : Error<"duplicate asm qualifier '%0'">;
}

let CategoryName = "Parse Issue" in {

def ext_empty_translation_unit : Extension<
  "ISO C requires a translation unit to contain at least one declaration">,
  InGroup<DiagGroup<"empty-translation-unit">>;
def warn_cxx98_compat_top_level_semi : Warning<
  "extra ';' outside of a function is incompatible with C++98">,
  InGroup<CXX98CompatExtraSemi>, DefaultIgnore;
def ext_extra_semi : Extension<
  "extra ';' %select{"
  "outside of a function|"
````
- **L25 EN**: Blank line separating nearby declarations or logic blocks.
  **L25 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L26 EN**: Assigns a TableGen property that affects following records or inherited fields: `let CategoryName = "Inline Assembly Issue" in {`.
  **L26 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let CategoryName = "Inline Assembly Issue" in {`。
- **L27 EN**: Declares TableGen def record `err_asm_empty`.
  **L27 CN**: 声明 TableGen def 记录 `err_asm_empty`。
- **L28 EN**: Declares TableGen def record `err_inline_ms_asm_parsing`.
  **L28 CN**: 声明 TableGen def 记录 `err_inline_ms_asm_parsing`。
- **L29 EN**: Declares TableGen def record `err_msasm_unsupported_arch`.
  **L29 CN**: 声明 TableGen def 记录 `err_msasm_unsupported_arch`。
- **L30 EN**: Adds a standalone statement or declaration: `"unsupported architecture '%0' for MS-style inline assembly">;`.
  **L30 CN**: 添加一条独立语句或声明：`"unsupported architecture '%0' for MS-style inline assembly">;`。
- **L31 EN**: Declares TableGen def record `err_msasm_unable_to_create_target`.
  **L31 CN**: 声明 TableGen def 记录 `err_msasm_unable_to_create_target`。
- **L32 EN**: Adds a standalone statement or declaration: `"MS-style inline assembly is not available: %0">;`.
  **L32 CN**: 添加一条独立语句或声明：`"MS-style inline assembly is not available: %0">;`。
- **L33 EN**: Declares TableGen def record `err_gnu_inline_asm_disabled`.
  **L33 CN**: 声明 TableGen def 记录 `err_gnu_inline_asm_disabled`。
- **L34 EN**: Adds a standalone statement or declaration: `"GNU-style inline assembly is disabled">;`.
  **L34 CN**: 添加一条独立语句或声明：`"GNU-style inline assembly is disabled">;`。
- **L35 EN**: Declares TableGen def record `err_asm_duplicate_qual`.
  **L35 CN**: 声明 TableGen def 记录 `err_asm_duplicate_qual`。
- **L36 EN**: Closes the current lexical scope or compound statement.
  **L36 CN**: 结束当前词法作用域或复合语句块。
- **L37 EN**: Blank line separating nearby declarations or logic blocks.
  **L37 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L38 EN**: Assigns a TableGen property that affects following records or inherited fields: `let CategoryName = "Parse Issue" in {`.
  **L38 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let CategoryName = "Parse Issue" in {`。
- **L39 EN**: Blank line separating nearby declarations or logic blocks.
  **L39 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L40 EN**: Declares TableGen def record `ext_empty_translation_unit`.
  **L40 CN**: 声明 TableGen def 记录 `ext_empty_translation_unit`。
- **L41 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"ISO C requires a translation unit to contain at least one declaration">,`.
  **L41 CN**: 继续一个多行参数列表、初始化器或聚合项：`"ISO C requires a translation unit to contain at least one declaration">,`。
- **L42 EN**: Adds a standalone statement or declaration: `InGroup<DiagGroup<"empty-translation-unit">>;`.
  **L42 CN**: 添加一条独立语句或声明：`InGroup<DiagGroup<"empty-translation-unit">>;`。
- **L43 EN**: Declares TableGen def record `warn_cxx98_compat_top_level_semi`.
  **L43 CN**: 声明 TableGen def 记录 `warn_cxx98_compat_top_level_semi`。
- **L44 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"extra ';' outside of a function is incompatible with C++98">,`.
  **L44 CN**: 继续一个多行参数列表、初始化器或聚合项：`"extra ';' outside of a function is incompatible with C++98">,`。
- **L45 EN**: Adds a standalone statement or declaration: `InGroup<CXX98CompatExtraSemi>, DefaultIgnore;`.
  **L45 CN**: 添加一条独立语句或声明：`InGroup<CXX98CompatExtraSemi>, DefaultIgnore;`。
- **L46 EN**: Declares TableGen def record `ext_extra_semi`.
  **L46 CN**: 声明 TableGen def 记录 `ext_extra_semi`。
- **L47 EN**: Continues the surrounding expression or declaration: `"extra ';' %select{"`.
  **L47 CN**: 继续构造周围的表达式或声明：`"extra ';' %select{"`。
- **L48 EN**: Continues the surrounding expression or declaration: `"outside of a function|"`.
  **L48 CN**: 继续构造周围的表达式或声明：`"outside of a function|"`。

### Lines 49-72

````tablegen
  "inside a %1|"
  "inside instance variable list|"
  "after member function definition}0">,
  InGroup<ExtraSemi>;
def ext_extra_semi_cxx11 : Extension<
  "extra ';' outside of a function is a C++11 extension">,
  InGroup<CXX11ExtraSemi>;
def warn_extra_semi_after_mem_fn_def : Warning<
  "extra ';' after member function definition">,
  InGroup<ExtraSemi>, DefaultIgnore;
def warn_null_statement : Warning<
  "empty expression statement has no effect; "
  "remove unnecessary ';' to silence this warning">,
  InGroup<ExtraSemiStmt>, DefaultIgnore;

def warn_misleading_indentation : Warning<
  "misleading indentation; statement is not part of "
  "the previous '%select{if|else|for|while}0'">,
  InGroup<MisleadingIndentation>, DefaultIgnore;
def note_previous_statement : Note<
  "previous statement is here">;

def subst_compound_token_kind : TextSubstitution<
  "%select{%1 and |}0%2 tokens "
````
- **L49 EN**: Continues the surrounding expression or declaration: `"inside a %1|"`.
  **L49 CN**: 继续构造周围的表达式或声明：`"inside a %1|"`。
- **L50 EN**: Continues the surrounding expression or declaration: `"inside instance variable list|"`.
  **L50 CN**: 继续构造周围的表达式或声明：`"inside instance variable list|"`。
- **L51 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"after member function definition}0">,`.
  **L51 CN**: 继续一个多行参数列表、初始化器或聚合项：`"after member function definition}0">,`。
- **L52 EN**: Adds a standalone statement or declaration: `InGroup<ExtraSemi>;`.
  **L52 CN**: 添加一条独立语句或声明：`InGroup<ExtraSemi>;`。
- **L53 EN**: Declares TableGen def record `ext_extra_semi_cxx11`.
  **L53 CN**: 声明 TableGen def 记录 `ext_extra_semi_cxx11`。
- **L54 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"extra ';' outside of a function is a C++11 extension">,`.
  **L54 CN**: 继续一个多行参数列表、初始化器或聚合项：`"extra ';' outside of a function is a C++11 extension">,`。
- **L55 EN**: Adds a standalone statement or declaration: `InGroup<CXX11ExtraSemi>;`.
  **L55 CN**: 添加一条独立语句或声明：`InGroup<CXX11ExtraSemi>;`。
- **L56 EN**: Declares TableGen def record `warn_extra_semi_after_mem_fn_def`.
  **L56 CN**: 声明 TableGen def 记录 `warn_extra_semi_after_mem_fn_def`。
- **L57 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"extra ';' after member function definition">,`.
  **L57 CN**: 继续一个多行参数列表、初始化器或聚合项：`"extra ';' after member function definition">,`。
- **L58 EN**: Adds a standalone statement or declaration: `InGroup<ExtraSemi>, DefaultIgnore;`.
  **L58 CN**: 添加一条独立语句或声明：`InGroup<ExtraSemi>, DefaultIgnore;`。
- **L59 EN**: Declares TableGen def record `warn_null_statement`.
  **L59 CN**: 声明 TableGen def 记录 `warn_null_statement`。
- **L60 EN**: Continues the surrounding expression or declaration: `"empty expression statement has no effect; "`.
  **L60 CN**: 继续构造周围的表达式或声明：`"empty expression statement has no effect; "`。
- **L61 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"remove unnecessary ';' to silence this warning">,`.
  **L61 CN**: 继续一个多行参数列表、初始化器或聚合项：`"remove unnecessary ';' to silence this warning">,`。
- **L62 EN**: Adds a standalone statement or declaration: `InGroup<ExtraSemiStmt>, DefaultIgnore;`.
  **L62 CN**: 添加一条独立语句或声明：`InGroup<ExtraSemiStmt>, DefaultIgnore;`。
- **L63 EN**: Blank line separating nearby declarations or logic blocks.
  **L63 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L64 EN**: Declares TableGen def record `warn_misleading_indentation`.
  **L64 CN**: 声明 TableGen def 记录 `warn_misleading_indentation`。
- **L65 EN**: Continues the surrounding expression or declaration: `"misleading indentation; statement is not part of "`.
  **L65 CN**: 继续构造周围的表达式或声明：`"misleading indentation; statement is not part of "`。
- **L66 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"the previous '%select{if|else|for|while}0'">,`.
  **L66 CN**: 继续一个多行参数列表、初始化器或聚合项：`"the previous '%select{if|else|for|while}0'">,`。
- **L67 EN**: Adds a standalone statement or declaration: `InGroup<MisleadingIndentation>, DefaultIgnore;`.
  **L67 CN**: 添加一条独立语句或声明：`InGroup<MisleadingIndentation>, DefaultIgnore;`。
- **L68 EN**: Declares TableGen def record `note_previous_statement`.
  **L68 CN**: 声明 TableGen def 记录 `note_previous_statement`。
- **L69 EN**: Adds a standalone statement or declaration: `"previous statement is here">;`.
  **L69 CN**: 添加一条独立语句或声明：`"previous statement is here">;`。
- **L70 EN**: Blank line separating nearby declarations or logic blocks.
  **L70 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L71 EN**: Declares TableGen def record `subst_compound_token_kind`.
  **L71 CN**: 声明 TableGen def 记录 `subst_compound_token_kind`。
- **L72 EN**: Continues the surrounding expression or declaration: `"%select{%1 and |}0%2 tokens "`.
  **L72 CN**: 继续构造周围的表达式或声明：`"%select{%1 and |}0%2 tokens "`。

### Lines 73-96

````tablegen
  "%select{introducing statement expression|terminating statement expression|"
  "introducing attribute|terminating attribute|"
  "forming pointer to member type}3">;
def warn_compound_token_split_by_macro : Warning<
  "%sub{subst_compound_token_kind}0,1,2,3 appear in different "
  "macro expansion contexts">, InGroup<CompoundTokenSplitByMacro>;
def note_compound_token_split_second_token_here : Note<
  "%select{|second }0%1 token is here">;
def warn_compound_token_split_by_whitespace : Warning<
  "%sub{subst_compound_token_kind}0,1,2,3 are separated by whitespace">,
  InGroup<CompoundTokenSplitBySpace>, DefaultIgnore;

def ext_thread_before : Extension<"'__thread' before '%0'">;
def ext_keyword_as_ident : ExtWarn<
  "keyword '%0' will be made available as an identifier "
  "%select{here|for the remainder of the translation unit}1">,
  InGroup<KeywordCompat>;

def ext_nullability : Extension<
  "type nullability specifier %0 is a Clang extension">,
  InGroup<DiagGroup<"nullability-extension">>;

def err_empty_enum : Error<"use of empty enum">;

````
- **L73 EN**: Continues the surrounding expression or declaration: `"%select{introducing statement expression|terminating statement expression|"`.
  **L73 CN**: 继续构造周围的表达式或声明：`"%select{introducing statement expression|terminating statement expression|"`。
- **L74 EN**: Continues the surrounding expression or declaration: `"introducing attribute|terminating attribute|"`.
  **L74 CN**: 继续构造周围的表达式或声明：`"introducing attribute|terminating attribute|"`。
- **L75 EN**: Adds a standalone statement or declaration: `"forming pointer to member type}3">;`.
  **L75 CN**: 添加一条独立语句或声明：`"forming pointer to member type}3">;`。
- **L76 EN**: Declares TableGen def record `warn_compound_token_split_by_macro`.
  **L76 CN**: 声明 TableGen def 记录 `warn_compound_token_split_by_macro`。
- **L77 EN**: Continues the surrounding expression or declaration: `"%sub{subst_compound_token_kind}0,1,2,3 appear in different "`.
  **L77 CN**: 继续构造周围的表达式或声明：`"%sub{subst_compound_token_kind}0,1,2,3 appear in different "`。
- **L78 EN**: Adds a standalone statement or declaration: `"macro expansion contexts">, InGroup<CompoundTokenSplitByMacro>;`.
  **L78 CN**: 添加一条独立语句或声明：`"macro expansion contexts">, InGroup<CompoundTokenSplitByMacro>;`。
- **L79 EN**: Declares TableGen def record `note_compound_token_split_second_token_here`.
  **L79 CN**: 声明 TableGen def 记录 `note_compound_token_split_second_token_here`。
- **L80 EN**: Adds a standalone statement or declaration: `"%select{|second }0%1 token is here">;`.
  **L80 CN**: 添加一条独立语句或声明：`"%select{|second }0%1 token is here">;`。
- **L81 EN**: Declares TableGen def record `warn_compound_token_split_by_whitespace`.
  **L81 CN**: 声明 TableGen def 记录 `warn_compound_token_split_by_whitespace`。
- **L82 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"%sub{subst_compound_token_kind}0,1,2,3 are separated by whitespace">,`.
  **L82 CN**: 继续一个多行参数列表、初始化器或聚合项：`"%sub{subst_compound_token_kind}0,1,2,3 are separated by whitespace">,`。
- **L83 EN**: Adds a standalone statement or declaration: `InGroup<CompoundTokenSplitBySpace>, DefaultIgnore;`.
  **L83 CN**: 添加一条独立语句或声明：`InGroup<CompoundTokenSplitBySpace>, DefaultIgnore;`。
- **L84 EN**: Blank line separating nearby declarations or logic blocks.
  **L84 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L85 EN**: Declares TableGen def record `ext_thread_before`.
  **L85 CN**: 声明 TableGen def 记录 `ext_thread_before`。
- **L86 EN**: Declares TableGen def record `ext_keyword_as_ident`.
  **L86 CN**: 声明 TableGen def 记录 `ext_keyword_as_ident`。
- **L87 EN**: Continues the surrounding expression or declaration: `"keyword '%0' will be made available as an identifier "`.
  **L87 CN**: 继续构造周围的表达式或声明：`"keyword '%0' will be made available as an identifier "`。
- **L88 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"%select{here|for the remainder of the translation unit}1">,`.
  **L88 CN**: 继续一个多行参数列表、初始化器或聚合项：`"%select{here|for the remainder of the translation unit}1">,`。
- **L89 EN**: Adds a standalone statement or declaration: `InGroup<KeywordCompat>;`.
  **L89 CN**: 添加一条独立语句或声明：`InGroup<KeywordCompat>;`。
- **L90 EN**: Blank line separating nearby declarations or logic blocks.
  **L90 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L91 EN**: Declares TableGen def record `ext_nullability`.
  **L91 CN**: 声明 TableGen def 记录 `ext_nullability`。
- **L92 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"type nullability specifier %0 is a Clang extension">,`.
  **L92 CN**: 继续一个多行参数列表、初始化器或聚合项：`"type nullability specifier %0 is a Clang extension">,`。
- **L93 EN**: Adds a standalone statement or declaration: `InGroup<DiagGroup<"nullability-extension">>;`.
  **L93 CN**: 添加一条独立语句或声明：`InGroup<DiagGroup<"nullability-extension">>;`。
- **L94 EN**: Blank line separating nearby declarations or logic blocks.
  **L94 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L95 EN**: Declares TableGen def record `err_empty_enum`.
  **L95 CN**: 声明 TableGen def 记录 `err_empty_enum`。
- **L96 EN**: Blank line separating nearby declarations or logic blocks.
  **L96 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 97-120

````tablegen
def ext_ident_list_in_param : Extension<
  "type-less parameter names in function declaration">;
def ext_c99_variable_decl_in_for_loop : Extension<
  "variable declaration in for loop is a C99-specific feature">, InGroup<C99>;
def ext_c99_compound_literal : Extension<
  "compound literals are a C99-specific feature">, InGroup<C99>;
def ext_enumerator_list_comma_c : Extension<
  "commas at the end of enumerator lists are a C99-specific "
  "feature">, InGroup<C99>;
def ext_enumerator_list_comma_cxx : Extension<
  "commas at the end of enumerator lists are a C++11 extension">,
  InGroup<CXX11>;
def warn_cxx98_compat_enumerator_list_comma : Warning<
  "commas at the end of enumerator lists are incompatible with C++98">,
  InGroup<CXX98CompatPedantic>, DefaultIgnore;
def err_enumerator_list_missing_comma : Error<
  "missing ',' between enumerators">;
def err_enumerator_unnamed_no_def : Error<
  "unnamed enumeration must be a definition">;
def ext_ms_c_enum_fixed_underlying_type : Extension<
  "enumeration types with a fixed underlying type are a Microsoft extension">,
  InGroup<MicrosoftFixedEnum>;
def ext_ms_c_empty_enum_type : Extension<
  "empty enumeration types are a Microsoft extension">,
````
- **L97 EN**: Declares TableGen def record `ext_ident_list_in_param`.
  **L97 CN**: 声明 TableGen def 记录 `ext_ident_list_in_param`。
- **L98 EN**: Adds a standalone statement or declaration: `"type-less parameter names in function declaration">;`.
  **L98 CN**: 添加一条独立语句或声明：`"type-less parameter names in function declaration">;`。
- **L99 EN**: Declares TableGen def record `ext_c99_variable_decl_in_for_loop`.
  **L99 CN**: 声明 TableGen def 记录 `ext_c99_variable_decl_in_for_loop`。
- **L100 EN**: Adds a standalone statement or declaration: `"variable declaration in for loop is a C99-specific feature">, InGroup<C99>;`.
  **L100 CN**: 添加一条独立语句或声明：`"variable declaration in for loop is a C99-specific feature">, InGroup<C99>;`。
- **L101 EN**: Declares TableGen def record `ext_c99_compound_literal`.
  **L101 CN**: 声明 TableGen def 记录 `ext_c99_compound_literal`。
- **L102 EN**: Adds a standalone statement or declaration: `"compound literals are a C99-specific feature">, InGroup<C99>;`.
  **L102 CN**: 添加一条独立语句或声明：`"compound literals are a C99-specific feature">, InGroup<C99>;`。
- **L103 EN**: Declares TableGen def record `ext_enumerator_list_comma_c`.
  **L103 CN**: 声明 TableGen def 记录 `ext_enumerator_list_comma_c`。
- **L104 EN**: Continues the surrounding expression or declaration: `"commas at the end of enumerator lists are a C99-specific "`.
  **L104 CN**: 继续构造周围的表达式或声明：`"commas at the end of enumerator lists are a C99-specific "`。
- **L105 EN**: Adds a standalone statement or declaration: `"feature">, InGroup<C99>;`.
  **L105 CN**: 添加一条独立语句或声明：`"feature">, InGroup<C99>;`。
- **L106 EN**: Declares TableGen def record `ext_enumerator_list_comma_cxx`.
  **L106 CN**: 声明 TableGen def 记录 `ext_enumerator_list_comma_cxx`。
- **L107 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"commas at the end of enumerator lists are a C++11 extension">,`.
  **L107 CN**: 继续一个多行参数列表、初始化器或聚合项：`"commas at the end of enumerator lists are a C++11 extension">,`。
- **L108 EN**: Adds a standalone statement or declaration: `InGroup<CXX11>;`.
  **L108 CN**: 添加一条独立语句或声明：`InGroup<CXX11>;`。
- **L109 EN**: Declares TableGen def record `warn_cxx98_compat_enumerator_list_comma`.
  **L109 CN**: 声明 TableGen def 记录 `warn_cxx98_compat_enumerator_list_comma`。
- **L110 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"commas at the end of enumerator lists are incompatible with C++98">,`.
  **L110 CN**: 继续一个多行参数列表、初始化器或聚合项：`"commas at the end of enumerator lists are incompatible with C++98">,`。
- **L111 EN**: Adds a standalone statement or declaration: `InGroup<CXX98CompatPedantic>, DefaultIgnore;`.
  **L111 CN**: 添加一条独立语句或声明：`InGroup<CXX98CompatPedantic>, DefaultIgnore;`。
- **L112 EN**: Declares TableGen def record `err_enumerator_list_missing_comma`.
  **L112 CN**: 声明 TableGen def 记录 `err_enumerator_list_missing_comma`。
- **L113 EN**: Adds a standalone statement or declaration: `"missing ',' between enumerators">;`.
  **L113 CN**: 添加一条独立语句或声明：`"missing ',' between enumerators">;`。
- **L114 EN**: Declares TableGen def record `err_enumerator_unnamed_no_def`.
  **L114 CN**: 声明 TableGen def 记录 `err_enumerator_unnamed_no_def`。
- **L115 EN**: Adds a standalone statement or declaration: `"unnamed enumeration must be a definition">;`.
  **L115 CN**: 添加一条独立语句或声明：`"unnamed enumeration must be a definition">;`。
- **L116 EN**: Declares TableGen def record `ext_ms_c_enum_fixed_underlying_type`.
  **L116 CN**: 声明 TableGen def 记录 `ext_ms_c_enum_fixed_underlying_type`。
- **L117 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"enumeration types with a fixed underlying type are a Microsoft extension">,`.
  **L117 CN**: 继续一个多行参数列表、初始化器或聚合项：`"enumeration types with a fixed underlying type are a Microsoft extension">,`。
- **L118 EN**: Adds a standalone statement or declaration: `InGroup<MicrosoftFixedEnum>;`.
  **L118 CN**: 添加一条独立语句或声明：`InGroup<MicrosoftFixedEnum>;`。
- **L119 EN**: Declares TableGen def record `ext_ms_c_empty_enum_type`.
  **L119 CN**: 声明 TableGen def 记录 `ext_ms_c_empty_enum_type`。
- **L120 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"empty enumeration types are a Microsoft extension">,`.
  **L120 CN**: 继续一个多行参数列表、初始化器或聚合项：`"empty enumeration types are a Microsoft extension">,`。

### Lines 121-144

````tablegen
  InGroup<MicrosoftEmptyEnum>;
def ext_c23_enum_fixed_underlying_type : Extension<
  "enumeration types with a fixed underlying type are a C23 extension">,
  InGroup<C23>;
def warn_c17_compat_enum_fixed_underlying_type : Warning<
  "enumeration types with a fixed underlying type are incompatible with C standards before C23">,
  DefaultIgnore, InGroup<CPre23Compat>;
def ext_enum_base_in_type_specifier : ExtWarn<
  "non-defining declaration of enumeration with a fixed underlying type is "
  "only permitted as a standalone declaration"
  "%select{|; missing list of enumerators?}0">,
  InGroup<DiagGroup<"elaborated-enum-base">>, DefaultError;
def ext_elaborated_enum_class : ExtWarn<
  "reference to enumeration must use 'enum' not 'enum %select{struct|class}0'">,
  InGroup<DiagGroup<"elaborated-enum-class">>, DefaultError;
def err_scoped_enum_missing_identifier : Error<
  "scoped enumeration requires a name">;
def ext_scoped_enum : ExtWarn<
  "scoped enumerations are a C++11 extension">, InGroup<CXX11>;
def warn_cxx98_compat_scoped_enum : Warning<
  "scoped enumerations are incompatible with C++98">,
  InGroup<CXX98Compat>, DefaultIgnore;
def err_anonymous_enum_bitfield : Error<
  "ISO C++ only allows ':' in member enumeration declaration to introduce "
````
- **L121 EN**: Adds a standalone statement or declaration: `InGroup<MicrosoftEmptyEnum>;`.
  **L121 CN**: 添加一条独立语句或声明：`InGroup<MicrosoftEmptyEnum>;`。
- **L122 EN**: Declares TableGen def record `ext_c23_enum_fixed_underlying_type`.
  **L122 CN**: 声明 TableGen def 记录 `ext_c23_enum_fixed_underlying_type`。
- **L123 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"enumeration types with a fixed underlying type are a C23 extension">,`.
  **L123 CN**: 继续一个多行参数列表、初始化器或聚合项：`"enumeration types with a fixed underlying type are a C23 extension">,`。
- **L124 EN**: Adds a standalone statement or declaration: `InGroup<C23>;`.
  **L124 CN**: 添加一条独立语句或声明：`InGroup<C23>;`。
- **L125 EN**: Declares TableGen def record `warn_c17_compat_enum_fixed_underlying_type`.
  **L125 CN**: 声明 TableGen def 记录 `warn_c17_compat_enum_fixed_underlying_type`。
- **L126 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"enumeration types with a fixed underlying type are incompatible with C standards before C23">,`.
  **L126 CN**: 继续一个多行参数列表、初始化器或聚合项：`"enumeration types with a fixed underlying type are incompatible with C standards before C23">,`。
- **L127 EN**: Adds a standalone statement or declaration: `DefaultIgnore, InGroup<CPre23Compat>;`.
  **L127 CN**: 添加一条独立语句或声明：`DefaultIgnore, InGroup<CPre23Compat>;`。
- **L128 EN**: Declares TableGen def record `ext_enum_base_in_type_specifier`.
  **L128 CN**: 声明 TableGen def 记录 `ext_enum_base_in_type_specifier`。
- **L129 EN**: Continues the surrounding expression or declaration: `"non-defining declaration of enumeration with a fixed underlying type is "`.
  **L129 CN**: 继续构造周围的表达式或声明：`"non-defining declaration of enumeration with a fixed underlying type is "`。
- **L130 EN**: Continues the surrounding expression or declaration: `"only permitted as a standalone declaration"`.
  **L130 CN**: 继续构造周围的表达式或声明：`"only permitted as a standalone declaration"`。
- **L131 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"%select{|; missing list of enumerators?}0">,`.
  **L131 CN**: 继续一个多行参数列表、初始化器或聚合项：`"%select{|; missing list of enumerators?}0">,`。
- **L132 EN**: Adds a standalone statement or declaration: `InGroup<DiagGroup<"elaborated-enum-base">>, DefaultError;`.
  **L132 CN**: 添加一条独立语句或声明：`InGroup<DiagGroup<"elaborated-enum-base">>, DefaultError;`。
- **L133 EN**: Declares TableGen def record `ext_elaborated_enum_class`.
  **L133 CN**: 声明 TableGen def 记录 `ext_elaborated_enum_class`。
- **L134 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"reference to enumeration must use 'enum' not 'enum %select{struct|class}0'">,`.
  **L134 CN**: 继续一个多行参数列表、初始化器或聚合项：`"reference to enumeration must use 'enum' not 'enum %select{struct|class}0'">,`。
- **L135 EN**: Adds a standalone statement or declaration: `InGroup<DiagGroup<"elaborated-enum-class">>, DefaultError;`.
  **L135 CN**: 添加一条独立语句或声明：`InGroup<DiagGroup<"elaborated-enum-class">>, DefaultError;`。
- **L136 EN**: Declares TableGen def record `err_scoped_enum_missing_identifier`.
  **L136 CN**: 声明 TableGen def 记录 `err_scoped_enum_missing_identifier`。
- **L137 EN**: Adds a standalone statement or declaration: `"scoped enumeration requires a name">;`.
  **L137 CN**: 添加一条独立语句或声明：`"scoped enumeration requires a name">;`。
- **L138 EN**: Declares TableGen def record `ext_scoped_enum`.
  **L138 CN**: 声明 TableGen def 记录 `ext_scoped_enum`。
- **L139 EN**: Adds a standalone statement or declaration: `"scoped enumerations are a C++11 extension">, InGroup<CXX11>;`.
  **L139 CN**: 添加一条独立语句或声明：`"scoped enumerations are a C++11 extension">, InGroup<CXX11>;`。
- **L140 EN**: Declares TableGen def record `warn_cxx98_compat_scoped_enum`.
  **L140 CN**: 声明 TableGen def 记录 `warn_cxx98_compat_scoped_enum`。
- **L141 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"scoped enumerations are incompatible with C++98">,`.
  **L141 CN**: 继续一个多行参数列表、初始化器或聚合项：`"scoped enumerations are incompatible with C++98">,`。
- **L142 EN**: Adds a standalone statement or declaration: `InGroup<CXX98Compat>, DefaultIgnore;`.
  **L142 CN**: 添加一条独立语句或声明：`InGroup<CXX98Compat>, DefaultIgnore;`。
- **L143 EN**: Declares TableGen def record `err_anonymous_enum_bitfield`.
  **L143 CN**: 声明 TableGen def 记录 `err_anonymous_enum_bitfield`。
- **L144 EN**: Continues the surrounding expression or declaration: `"ISO C++ only allows ':' in member enumeration declaration to introduce "`.
  **L144 CN**: 继续构造周围的表达式或声明：`"ISO C++ only allows ':' in member enumeration declaration to introduce "`。

### Lines 145-168

````tablegen
  "a fixed underlying type, not an anonymous bit-field">;

def warn_cxx98_compat_alignof : Warning<
  "alignof expressions are incompatible with C++98">,
  InGroup<CXX98Compat>, DefaultIgnore;
def ext_alignof_expr : ExtWarn<
  "%0 applied to an expression is a GNU extension">, InGroup<GNUAlignofExpression>;
def err_lambda_after_delete : Error<
  "'[]' after delete interpreted as 'delete[]'; add parentheses to treat this as a lambda-expression">;

def warn_microsoft_dependent_exists : Warning<
  "dependent %select{__if_not_exists|__if_exists}0 declarations are ignored">,
  InGroup<DiagGroup<"microsoft-exists">>;
def warn_microsoft_qualifiers_ignored : Warning<
  "qualifiers after comma in declarator list are ignored">,
  InGroup<IgnoredAttributes>;

def err_duplicate_default_assoc : Error<
  "duplicate default generic association">;
def note_previous_default_assoc : Note<
  "previous default generic association is here">;
def ext_c2y_generic_with_type_arg : Extension<
  "passing a type argument as the first operand to '_Generic' is a C2y "
  "extension">, InGroup<C2y>;
````
- **L145 EN**: Adds a standalone statement or declaration: `"a fixed underlying type, not an anonymous bit-field">;`.
  **L145 CN**: 添加一条独立语句或声明：`"a fixed underlying type, not an anonymous bit-field">;`。
- **L146 EN**: Blank line separating nearby declarations or logic blocks.
  **L146 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L147 EN**: Declares TableGen def record `warn_cxx98_compat_alignof`.
  **L147 CN**: 声明 TableGen def 记录 `warn_cxx98_compat_alignof`。
- **L148 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"alignof expressions are incompatible with C++98">,`.
  **L148 CN**: 继续一个多行参数列表、初始化器或聚合项：`"alignof expressions are incompatible with C++98">,`。
- **L149 EN**: Adds a standalone statement or declaration: `InGroup<CXX98Compat>, DefaultIgnore;`.
  **L149 CN**: 添加一条独立语句或声明：`InGroup<CXX98Compat>, DefaultIgnore;`。
- **L150 EN**: Declares TableGen def record `ext_alignof_expr`.
  **L150 CN**: 声明 TableGen def 记录 `ext_alignof_expr`。
- **L151 EN**: Adds a standalone statement or declaration: `"%0 applied to an expression is a GNU extension">, InGroup<GNUAlignofExpression>;`.
  **L151 CN**: 添加一条独立语句或声明：`"%0 applied to an expression is a GNU extension">, InGroup<GNUAlignofExpression>;`。
- **L152 EN**: Declares TableGen def record `err_lambda_after_delete`.
  **L152 CN**: 声明 TableGen def 记录 `err_lambda_after_delete`。
- **L153 EN**: Adds a standalone statement or declaration: `"'[]' after delete interpreted as 'delete[]'; add parentheses to treat this as a lambda-expression">;`.
  **L153 CN**: 添加一条独立语句或声明：`"'[]' after delete interpreted as 'delete[]'; add parentheses to treat this as a lambda-expression">;`。
- **L154 EN**: Blank line separating nearby declarations or logic blocks.
  **L154 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L155 EN**: Declares TableGen def record `warn_microsoft_dependent_exists`.
  **L155 CN**: 声明 TableGen def 记录 `warn_microsoft_dependent_exists`。
- **L156 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"dependent %select{__if_not_exists|__if_exists}0 declarations are ignored">,`.
  **L156 CN**: 继续一个多行参数列表、初始化器或聚合项：`"dependent %select{__if_not_exists|__if_exists}0 declarations are ignored">,`。
- **L157 EN**: Adds a standalone statement or declaration: `InGroup<DiagGroup<"microsoft-exists">>;`.
  **L157 CN**: 添加一条独立语句或声明：`InGroup<DiagGroup<"microsoft-exists">>;`。
- **L158 EN**: Declares TableGen def record `warn_microsoft_qualifiers_ignored`.
  **L158 CN**: 声明 TableGen def 记录 `warn_microsoft_qualifiers_ignored`。
- **L159 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"qualifiers after comma in declarator list are ignored">,`.
  **L159 CN**: 继续一个多行参数列表、初始化器或聚合项：`"qualifiers after comma in declarator list are ignored">,`。
- **L160 EN**: Adds a standalone statement or declaration: `InGroup<IgnoredAttributes>;`.
  **L160 CN**: 添加一条独立语句或声明：`InGroup<IgnoredAttributes>;`。
- **L161 EN**: Blank line separating nearby declarations or logic blocks.
  **L161 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L162 EN**: Declares TableGen def record `err_duplicate_default_assoc`.
  **L162 CN**: 声明 TableGen def 记录 `err_duplicate_default_assoc`。
- **L163 EN**: Adds a standalone statement or declaration: `"duplicate default generic association">;`.
  **L163 CN**: 添加一条独立语句或声明：`"duplicate default generic association">;`。
- **L164 EN**: Declares TableGen def record `note_previous_default_assoc`.
  **L164 CN**: 声明 TableGen def 记录 `note_previous_default_assoc`。
- **L165 EN**: Adds a standalone statement or declaration: `"previous default generic association is here">;`.
  **L165 CN**: 添加一条独立语句或声明：`"previous default generic association is here">;`。
- **L166 EN**: Declares TableGen def record `ext_c2y_generic_with_type_arg`.
  **L166 CN**: 声明 TableGen def 记录 `ext_c2y_generic_with_type_arg`。
- **L167 EN**: Continues the surrounding expression or declaration: `"passing a type argument as the first operand to '_Generic' is a C2y "`.
  **L167 CN**: 继续构造周围的表达式或声明：`"passing a type argument as the first operand to '_Generic' is a C2y "`。
- **L168 EN**: Adds a standalone statement or declaration: `"extension">, InGroup<C2y>;`.
  **L168 CN**: 添加一条独立语句或声明：`"extension">, InGroup<C2y>;`。

### Lines 169-192

````tablegen
def warn_c2y_compat_generic_with_type_arg : Warning<
  "passing a type argument as the first operand to '_Generic' is incompatible "
  "with C standards before C2y">, InGroup<CPre2yCompat>, DefaultIgnore;

def ext_c99_feature : Extension<
  "'%0' is a C99 extension">, InGroup<C99>;
def ext_c11_feature : Extension<
  "'%0' is a C11 extension">, InGroup<C11>;
def ext_c2y_feature : Extension<
  "'%0' is a C2y extension">, InGroup<C2y>;
def warn_c11_compat_keyword : Warning<
  "'%0' is incompatible with C standards before C11">,
  InGroup<CPre11Compat>, DefaultIgnore;
def warn_c23_compat_keyword : Warning<
 "'%0' is incompatible with C standards before C23">,
 InGroup<CPre23Compat>, DefaultIgnore;
def warn_c2y_compat_keyword : Warning<
  "'%0' is incompatible with C standards before C2y">,
  InGroup<CPre2yCompat>, DefaultIgnore;

def err_c11_noreturn_misplaced : Error<
  "'_Noreturn' keyword must precede function declarator">;

def ext_gnu_indirect_goto : Extension<
````
- **L169 EN**: Declares TableGen def record `warn_c2y_compat_generic_with_type_arg`.
  **L169 CN**: 声明 TableGen def 记录 `warn_c2y_compat_generic_with_type_arg`。
- **L170 EN**: Continues the surrounding expression or declaration: `"passing a type argument as the first operand to '_Generic' is incompatible "`.
  **L170 CN**: 继续构造周围的表达式或声明：`"passing a type argument as the first operand to '_Generic' is incompatible "`。
- **L171 EN**: Adds a standalone statement or declaration: `"with C standards before C2y">, InGroup<CPre2yCompat>, DefaultIgnore;`.
  **L171 CN**: 添加一条独立语句或声明：`"with C standards before C2y">, InGroup<CPre2yCompat>, DefaultIgnore;`。
- **L172 EN**: Blank line separating nearby declarations or logic blocks.
  **L172 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L173 EN**: Declares TableGen def record `ext_c99_feature`.
  **L173 CN**: 声明 TableGen def 记录 `ext_c99_feature`。
- **L174 EN**: Adds a standalone statement or declaration: `"'%0' is a C99 extension">, InGroup<C99>;`.
  **L174 CN**: 添加一条独立语句或声明：`"'%0' is a C99 extension">, InGroup<C99>;`。
- **L175 EN**: Declares TableGen def record `ext_c11_feature`.
  **L175 CN**: 声明 TableGen def 记录 `ext_c11_feature`。
- **L176 EN**: Adds a standalone statement or declaration: `"'%0' is a C11 extension">, InGroup<C11>;`.
  **L176 CN**: 添加一条独立语句或声明：`"'%0' is a C11 extension">, InGroup<C11>;`。
- **L177 EN**: Declares TableGen def record `ext_c2y_feature`.
  **L177 CN**: 声明 TableGen def 记录 `ext_c2y_feature`。
- **L178 EN**: Adds a standalone statement or declaration: `"'%0' is a C2y extension">, InGroup<C2y>;`.
  **L178 CN**: 添加一条独立语句或声明：`"'%0' is a C2y extension">, InGroup<C2y>;`。
- **L179 EN**: Declares TableGen def record `warn_c11_compat_keyword`.
  **L179 CN**: 声明 TableGen def 记录 `warn_c11_compat_keyword`。
- **L180 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'%0' is incompatible with C standards before C11">,`.
  **L180 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'%0' is incompatible with C standards before C11">,`。
- **L181 EN**: Adds a standalone statement or declaration: `InGroup<CPre11Compat>, DefaultIgnore;`.
  **L181 CN**: 添加一条独立语句或声明：`InGroup<CPre11Compat>, DefaultIgnore;`。
- **L182 EN**: Declares TableGen def record `warn_c23_compat_keyword`.
  **L182 CN**: 声明 TableGen def 记录 `warn_c23_compat_keyword`。
- **L183 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'%0' is incompatible with C standards before C23">,`.
  **L183 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'%0' is incompatible with C standards before C23">,`。
- **L184 EN**: Adds a standalone statement or declaration: `InGroup<CPre23Compat>, DefaultIgnore;`.
  **L184 CN**: 添加一条独立语句或声明：`InGroup<CPre23Compat>, DefaultIgnore;`。
- **L185 EN**: Declares TableGen def record `warn_c2y_compat_keyword`.
  **L185 CN**: 声明 TableGen def 记录 `warn_c2y_compat_keyword`。
- **L186 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'%0' is incompatible with C standards before C2y">,`.
  **L186 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'%0' is incompatible with C standards before C2y">,`。
- **L187 EN**: Adds a standalone statement or declaration: `InGroup<CPre2yCompat>, DefaultIgnore;`.
  **L187 CN**: 添加一条独立语句或声明：`InGroup<CPre2yCompat>, DefaultIgnore;`。
- **L188 EN**: Blank line separating nearby declarations or logic blocks.
  **L188 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L189 EN**: Declares TableGen def record `err_c11_noreturn_misplaced`.
  **L189 CN**: 声明 TableGen def 记录 `err_c11_noreturn_misplaced`。
- **L190 EN**: Adds a standalone statement or declaration: `"'_Noreturn' keyword must precede function declarator">;`.
  **L190 CN**: 添加一条独立语句或声明：`"'_Noreturn' keyword must precede function declarator">;`。
- **L191 EN**: Blank line separating nearby declarations or logic blocks.
  **L191 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L192 EN**: Declares TableGen def record `ext_gnu_indirect_goto`.
  **L192 CN**: 声明 TableGen def 记录 `ext_gnu_indirect_goto`。

### Lines 193-216

````tablegen
  "use of GNU indirect-goto extension">, InGroup<GNULabelsAsValue>;
def ext_gnu_address_of_label : Extension<
  "use of GNU address-of-label extension">, InGroup<GNULabelsAsValue>;
def err_stmtexpr_file_scope : Error<
  "statement expression not allowed at file scope">;
def ext_gnu_statement_expr : Extension<
  "use of GNU statement expression extension">, InGroup<GNUStatementExpression>;
def ext_gnu_statement_expr_macro : Extension<
  "use of GNU statement expression extension from macro expansion">,
  InGroup<GNUStatementExpressionFromMacroExpansion>;
def ext_gnu_conditional_expr : Extension<
  "use of GNU ?: conditional expression extension, omitting middle operand">, InGroup<GNUConditionalOmittedOperand>;
def ext_gnu_array_range : Extension<"use of GNU array range extension">,
  InGroup<GNUDesignator>;
def ext_gnu_missing_equal_designator : ExtWarn<
  "use of GNU 'missing =' extension in designator">,
  InGroup<GNUDesignator>;
def err_expected_equal_designator : Error<"expected '=' or another designator">;
def ext_gnu_old_style_field_designator : ExtWarn<
  "use of GNU old-style field designator extension">,
  InGroup<GNUDesignator>;
def ext_gnu_case_range : Extension<
  "case ranges are a GNU extension">, InGroup<GNUCaseRange>;
def warn_c23_compat_case_range : Warning<
````
- **L193 EN**: Adds a standalone statement or declaration: `"use of GNU indirect-goto extension">, InGroup<GNULabelsAsValue>;`.
  **L193 CN**: 添加一条独立语句或声明：`"use of GNU indirect-goto extension">, InGroup<GNULabelsAsValue>;`。
- **L194 EN**: Declares TableGen def record `ext_gnu_address_of_label`.
  **L194 CN**: 声明 TableGen def 记录 `ext_gnu_address_of_label`。
- **L195 EN**: Adds a standalone statement or declaration: `"use of GNU address-of-label extension">, InGroup<GNULabelsAsValue>;`.
  **L195 CN**: 添加一条独立语句或声明：`"use of GNU address-of-label extension">, InGroup<GNULabelsAsValue>;`。
- **L196 EN**: Declares TableGen def record `err_stmtexpr_file_scope`.
  **L196 CN**: 声明 TableGen def 记录 `err_stmtexpr_file_scope`。
- **L197 EN**: Adds a standalone statement or declaration: `"statement expression not allowed at file scope">;`.
  **L197 CN**: 添加一条独立语句或声明：`"statement expression not allowed at file scope">;`。
- **L198 EN**: Declares TableGen def record `ext_gnu_statement_expr`.
  **L198 CN**: 声明 TableGen def 记录 `ext_gnu_statement_expr`。
- **L199 EN**: Adds a standalone statement or declaration: `"use of GNU statement expression extension">, InGroup<GNUStatementExpression>;`.
  **L199 CN**: 添加一条独立语句或声明：`"use of GNU statement expression extension">, InGroup<GNUStatementExpression>;`。
- **L200 EN**: Declares TableGen def record `ext_gnu_statement_expr_macro`.
  **L200 CN**: 声明 TableGen def 记录 `ext_gnu_statement_expr_macro`。
- **L201 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"use of GNU statement expression extension from macro expansion">,`.
  **L201 CN**: 继续一个多行参数列表、初始化器或聚合项：`"use of GNU statement expression extension from macro expansion">,`。
- **L202 EN**: Adds a standalone statement or declaration: `InGroup<GNUStatementExpressionFromMacroExpansion>;`.
  **L202 CN**: 添加一条独立语句或声明：`InGroup<GNUStatementExpressionFromMacroExpansion>;`。
- **L203 EN**: Declares TableGen def record `ext_gnu_conditional_expr`.
  **L203 CN**: 声明 TableGen def 记录 `ext_gnu_conditional_expr`。
- **L204 EN**: Adds a standalone statement or declaration: `"use of GNU ?: conditional expression extension, omitting middle operand">, InGroup<GNUConditionalOmittedOperand>;`.
  **L204 CN**: 添加一条独立语句或声明：`"use of GNU ?: conditional expression extension, omitting middle operand">, InGroup<GNUConditionalOmittedOperand>;`。
- **L205 EN**: Declares TableGen def record `ext_gnu_array_range`.
  **L205 CN**: 声明 TableGen def 记录 `ext_gnu_array_range`。
- **L206 EN**: Adds a standalone statement or declaration: `InGroup<GNUDesignator>;`.
  **L206 CN**: 添加一条独立语句或声明：`InGroup<GNUDesignator>;`。
- **L207 EN**: Declares TableGen def record `ext_gnu_missing_equal_designator`.
  **L207 CN**: 声明 TableGen def 记录 `ext_gnu_missing_equal_designator`。
- **L208 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"use of GNU 'missing =' extension in designator">,`.
  **L208 CN**: 继续一个多行参数列表、初始化器或聚合项：`"use of GNU 'missing =' extension in designator">,`。
- **L209 EN**: Adds a standalone statement or declaration: `InGroup<GNUDesignator>;`.
  **L209 CN**: 添加一条独立语句或声明：`InGroup<GNUDesignator>;`。
- **L210 EN**: Declares TableGen def record `err_expected_equal_designator`.
  **L210 CN**: 声明 TableGen def 记录 `err_expected_equal_designator`。
- **L211 EN**: Declares TableGen def record `ext_gnu_old_style_field_designator`.
  **L211 CN**: 声明 TableGen def 记录 `ext_gnu_old_style_field_designator`。
- **L212 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"use of GNU old-style field designator extension">,`.
  **L212 CN**: 继续一个多行参数列表、初始化器或聚合项：`"use of GNU old-style field designator extension">,`。
- **L213 EN**: Adds a standalone statement or declaration: `InGroup<GNUDesignator>;`.
  **L213 CN**: 添加一条独立语句或声明：`InGroup<GNUDesignator>;`。
- **L214 EN**: Declares TableGen def record `ext_gnu_case_range`.
  **L214 CN**: 声明 TableGen def 记录 `ext_gnu_case_range`。
- **L215 EN**: Adds a standalone statement or declaration: `"case ranges are a GNU extension">, InGroup<GNUCaseRange>;`.
  **L215 CN**: 添加一条独立语句或声明：`"case ranges are a GNU extension">, InGroup<GNUCaseRange>;`。
- **L216 EN**: Declares TableGen def record `warn_c23_compat_case_range`.
  **L216 CN**: 声明 TableGen def 记录 `warn_c23_compat_case_range`。

### Lines 217-240

````tablegen
  "case ranges are incompatible with C standards before C2y">,
  DefaultIgnore, InGroup<CPre2yCompat>;
def ext_c2y_case_range : Extension<
  "case ranges are a C2y extension">, InGroup<C2y>;
def err_c2y_labeled_break_continue : Error<
  "named %select{'break'|'continue'}0 is only supported in C2y">;

// Generic errors.
def err_expected_expression : Error<"expected expression">;
def err_expected_type : Error<"expected a type">;
def err_expected_external_declaration : Error<"expected external declaration">;
def err_extraneous_closing_brace : Error<"extraneous closing brace ('}')">;
def err_expected_semi_declaration : Error<
  "expected ';' at end of declaration">;
def err_expected_semi_decl_list : Error<
  "expected ';' at end of declaration list">;
def ext_expected_semi_decl_list : ExtWarn<
  "expected ';' at end of declaration list">;
def err_expected_member_name_or_semi : Error<
  "expected member name or ';' after declaration specifiers">;
def err_function_declared_typedef : Error<
  "function definition declared 'typedef'">;
def err_at_defs_cxx : Error<"@defs is not supported in Objective-C++">;
def err_at_in_class : Error<"unexpected '@' in member specification">;
````
- **L217 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"case ranges are incompatible with C standards before C2y">,`.
  **L217 CN**: 继续一个多行参数列表、初始化器或聚合项：`"case ranges are incompatible with C standards before C2y">,`。
- **L218 EN**: Adds a standalone statement or declaration: `DefaultIgnore, InGroup<CPre2yCompat>;`.
  **L218 CN**: 添加一条独立语句或声明：`DefaultIgnore, InGroup<CPre2yCompat>;`。
- **L219 EN**: Declares TableGen def record `ext_c2y_case_range`.
  **L219 CN**: 声明 TableGen def 记录 `ext_c2y_case_range`。
- **L220 EN**: Adds a standalone statement or declaration: `"case ranges are a C2y extension">, InGroup<C2y>;`.
  **L220 CN**: 添加一条独立语句或声明：`"case ranges are a C2y extension">, InGroup<C2y>;`。
- **L221 EN**: Declares TableGen def record `err_c2y_labeled_break_continue`.
  **L221 CN**: 声明 TableGen def 记录 `err_c2y_labeled_break_continue`。
- **L222 EN**: Adds a standalone statement or declaration: `"named %select{'break'|'continue'}0 is only supported in C2y">;`.
  **L222 CN**: 添加一条独立语句或声明：`"named %select{'break'|'continue'}0 is only supported in C2y">;`。
- **L223 EN**: Blank line separating nearby declarations or logic blocks.
  **L223 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L224 EN**: Comment explains nearby logic, constraints, or intent: `Generic errors.`.
  **L224 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Generic errors.`。
- **L225 EN**: Declares TableGen def record `err_expected_expression`.
  **L225 CN**: 声明 TableGen def 记录 `err_expected_expression`。
- **L226 EN**: Declares TableGen def record `err_expected_type`.
  **L226 CN**: 声明 TableGen def 记录 `err_expected_type`。
- **L227 EN**: Declares TableGen def record `err_expected_external_declaration`.
  **L227 CN**: 声明 TableGen def 记录 `err_expected_external_declaration`。
- **L228 EN**: Declares TableGen def record `err_extraneous_closing_brace`.
  **L228 CN**: 声明 TableGen def 记录 `err_extraneous_closing_brace`。
- **L229 EN**: Declares TableGen def record `err_expected_semi_declaration`.
  **L229 CN**: 声明 TableGen def 记录 `err_expected_semi_declaration`。
- **L230 EN**: Adds a standalone statement or declaration: `"expected ';' at end of declaration">;`.
  **L230 CN**: 添加一条独立语句或声明：`"expected ';' at end of declaration">;`。
- **L231 EN**: Declares TableGen def record `err_expected_semi_decl_list`.
  **L231 CN**: 声明 TableGen def 记录 `err_expected_semi_decl_list`。
- **L232 EN**: Adds a standalone statement or declaration: `"expected ';' at end of declaration list">;`.
  **L232 CN**: 添加一条独立语句或声明：`"expected ';' at end of declaration list">;`。
- **L233 EN**: Declares TableGen def record `ext_expected_semi_decl_list`.
  **L233 CN**: 声明 TableGen def 记录 `ext_expected_semi_decl_list`。
- **L234 EN**: Adds a standalone statement or declaration: `"expected ';' at end of declaration list">;`.
  **L234 CN**: 添加一条独立语句或声明：`"expected ';' at end of declaration list">;`。
- **L235 EN**: Declares TableGen def record `err_expected_member_name_or_semi`.
  **L235 CN**: 声明 TableGen def 记录 `err_expected_member_name_or_semi`。
- **L236 EN**: Adds a standalone statement or declaration: `"expected member name or ';' after declaration specifiers">;`.
  **L236 CN**: 添加一条独立语句或声明：`"expected member name or ';' after declaration specifiers">;`。
- **L237 EN**: Declares TableGen def record `err_function_declared_typedef`.
  **L237 CN**: 声明 TableGen def 记录 `err_function_declared_typedef`。
- **L238 EN**: Adds a standalone statement or declaration: `"function definition declared 'typedef'">;`.
  **L238 CN**: 添加一条独立语句或声明：`"function definition declared 'typedef'">;`。
- **L239 EN**: Declares TableGen def record `err_at_defs_cxx`.
  **L239 CN**: 声明 TableGen def 记录 `err_at_defs_cxx`。
- **L240 EN**: Declares TableGen def record `err_at_in_class`.
  **L240 CN**: 声明 TableGen def 记录 `err_at_in_class`。

### Lines 241-264

````tablegen
def err_unexpected_semi : Error<"unexpected ';' before %0">;
def err_postfix_after_unary_requires_parens : Error<
  "expression cannot be followed by a postfix %0 operator; add parentheses">;
def err_unparenthesized_non_primary_expr_in_requires_clause : Error<
  "parentheses are required around this expression in a requires clause">;
def note_unparenthesized_non_primary_expr_in_requires_clause : Note<
  "parentheses are required around this expression in a requires clause">;

def err_expected_fn_body : Error<
  "expected function body after function declarator">;
def warn_attribute_on_function_definition : Warning<
  "GCC does not allow %0 attribute in this position on a function definition">,
  InGroup<GccCompat>;
def warn_gcc_attribute_location : Warning<
  "GCC does not allow an attribute in this position on a function declaration">,
  InGroup<GccCompat>;
def warn_gcc_variable_decl_in_for_loop : Warning<
  "GCC does not allow variable declarations in for loop initializers before "
  "C99">, InGroup<GccCompat>;
def warn_attribute_no_decl : Warning<
  "attribute %0 ignored, because it is not attached to a declaration">,
  InGroup<IgnoredAttributes>;
def err_ms_attributes_not_enabled : Error<
  "'__declspec' attributes are not enabled; use '-fdeclspec' or "
````
- **L241 EN**: Declares TableGen def record `err_unexpected_semi`.
  **L241 CN**: 声明 TableGen def 记录 `err_unexpected_semi`。
- **L242 EN**: Declares TableGen def record `err_postfix_after_unary_requires_parens`.
  **L242 CN**: 声明 TableGen def 记录 `err_postfix_after_unary_requires_parens`。
- **L243 EN**: Adds a standalone statement or declaration: `"expression cannot be followed by a postfix %0 operator; add parentheses">;`.
  **L243 CN**: 添加一条独立语句或声明：`"expression cannot be followed by a postfix %0 operator; add parentheses">;`。
- **L244 EN**: Declares TableGen def record `err_unparenthesized_non_primary_expr_in_requires_clause`.
  **L244 CN**: 声明 TableGen def 记录 `err_unparenthesized_non_primary_expr_in_requires_clause`。
- **L245 EN**: Adds a standalone statement or declaration: `"parentheses are required around this expression in a requires clause">;`.
  **L245 CN**: 添加一条独立语句或声明：`"parentheses are required around this expression in a requires clause">;`。
- **L246 EN**: Declares TableGen def record `note_unparenthesized_non_primary_expr_in_requires_clause`.
  **L246 CN**: 声明 TableGen def 记录 `note_unparenthesized_non_primary_expr_in_requires_clause`。
- **L247 EN**: Adds a standalone statement or declaration: `"parentheses are required around this expression in a requires clause">;`.
  **L247 CN**: 添加一条独立语句或声明：`"parentheses are required around this expression in a requires clause">;`。
- **L248 EN**: Blank line separating nearby declarations or logic blocks.
  **L248 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L249 EN**: Declares TableGen def record `err_expected_fn_body`.
  **L249 CN**: 声明 TableGen def 记录 `err_expected_fn_body`。
- **L250 EN**: Adds a standalone statement or declaration: `"expected function body after function declarator">;`.
  **L250 CN**: 添加一条独立语句或声明：`"expected function body after function declarator">;`。
- **L251 EN**: Declares TableGen def record `warn_attribute_on_function_definition`.
  **L251 CN**: 声明 TableGen def 记录 `warn_attribute_on_function_definition`。
- **L252 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"GCC does not allow %0 attribute in this position on a function definition">,`.
  **L252 CN**: 继续一个多行参数列表、初始化器或聚合项：`"GCC does not allow %0 attribute in this position on a function definition">,`。
- **L253 EN**: Adds a standalone statement or declaration: `InGroup<GccCompat>;`.
  **L253 CN**: 添加一条独立语句或声明：`InGroup<GccCompat>;`。
- **L254 EN**: Declares TableGen def record `warn_gcc_attribute_location`.
  **L254 CN**: 声明 TableGen def 记录 `warn_gcc_attribute_location`。
- **L255 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"GCC does not allow an attribute in this position on a function declaration">,`.
  **L255 CN**: 继续一个多行参数列表、初始化器或聚合项：`"GCC does not allow an attribute in this position on a function declaration">,`。
- **L256 EN**: Adds a standalone statement or declaration: `InGroup<GccCompat>;`.
  **L256 CN**: 添加一条独立语句或声明：`InGroup<GccCompat>;`。
- **L257 EN**: Declares TableGen def record `warn_gcc_variable_decl_in_for_loop`.
  **L257 CN**: 声明 TableGen def 记录 `warn_gcc_variable_decl_in_for_loop`。
- **L258 EN**: Continues the surrounding expression or declaration: `"GCC does not allow variable declarations in for loop initializers before "`.
  **L258 CN**: 继续构造周围的表达式或声明：`"GCC does not allow variable declarations in for loop initializers before "`。
- **L259 EN**: Adds a standalone statement or declaration: `"C99">, InGroup<GccCompat>;`.
  **L259 CN**: 添加一条独立语句或声明：`"C99">, InGroup<GccCompat>;`。
- **L260 EN**: Declares TableGen def record `warn_attribute_no_decl`.
  **L260 CN**: 声明 TableGen def 记录 `warn_attribute_no_decl`。
- **L261 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"attribute %0 ignored, because it is not attached to a declaration">,`.
  **L261 CN**: 继续一个多行参数列表、初始化器或聚合项：`"attribute %0 ignored, because it is not attached to a declaration">,`。
- **L262 EN**: Adds a standalone statement or declaration: `InGroup<IgnoredAttributes>;`.
  **L262 CN**: 添加一条独立语句或声明：`InGroup<IgnoredAttributes>;`。
- **L263 EN**: Declares TableGen def record `err_ms_attributes_not_enabled`.
  **L263 CN**: 声明 TableGen def 记录 `err_ms_attributes_not_enabled`。
- **L264 EN**: Applies compiler-specific attributes that refine ABI, layout, inlining, or diagnostics: `"'__declspec' attributes are not enabled; use '-fdeclspec' or "`.
  **L264 CN**: 应用编译器特定属性，以细化 ABI、布局、内联或诊断行为：`"'__declspec' attributes are not enabled; use '-fdeclspec' or "`。

### Lines 265-288

````tablegen
  "'-fms-extensions' to enable support for __declspec attributes">;
def err_expected_method_body : Error<"expected method body">;
def err_declspec_after_virtspec : Error<
  "'%0' qualifier may not appear after the virtual specifier '%1'">;
def err_invalid_token_after_toplevel_declarator : Error<
  "expected ';' after top level declarator">;
def err_invalid_token_after_declarator_suggest_equal : Error<
  "invalid %0 at end of declaration; did you mean '='?">;
def err_expected_statement : Error<"expected statement">;
def err_expected_lparen_after : Error<"expected '(' after '%0'">;
def err_expected_rparen_after : Error<"expected ')' after '%0'">;
def err_expected_punc : Error<"expected ')' or ',' after '%0'">;
def err_expected_less_after : Error<"expected '<' after '%0'">;
def err_expected_lbrace_in_compound_literal : Error<
  "expected '{' in compound literal">;
def err_expected_while : Error<"expected 'while' in do/while loop">;

def err_expected_semi_after_stmt : Error<"expected ';' after %0 statement">;
def err_expected_semi_after_expr : Error<"expected ';' after expression">;
def warn_attr_in_secondary_block : ExtWarn<
  "ISO C does not allow an attribute list to appear here">,
  InGroup<DiagGroup<"c-attribute-extension">>;
def err_extraneous_token_before_semi : Error<"extraneous '%0' before ';'">;

````
- **L265 EN**: Applies compiler-specific attributes that refine ABI, layout, inlining, or diagnostics: `"'-fms-extensions' to enable support for __declspec attributes">;`.
  **L265 CN**: 应用编译器特定属性，以细化 ABI、布局、内联或诊断行为：`"'-fms-extensions' to enable support for __declspec attributes">;`。
- **L266 EN**: Declares TableGen def record `err_expected_method_body`.
  **L266 CN**: 声明 TableGen def 记录 `err_expected_method_body`。
- **L267 EN**: Declares TableGen def record `err_declspec_after_virtspec`.
  **L267 CN**: 声明 TableGen def 记录 `err_declspec_after_virtspec`。
- **L268 EN**: Adds a standalone statement or declaration: `"'%0' qualifier may not appear after the virtual specifier '%1'">;`.
  **L268 CN**: 添加一条独立语句或声明：`"'%0' qualifier may not appear after the virtual specifier '%1'">;`。
- **L269 EN**: Declares TableGen def record `err_invalid_token_after_toplevel_declarator`.
  **L269 CN**: 声明 TableGen def 记录 `err_invalid_token_after_toplevel_declarator`。
- **L270 EN**: Adds a standalone statement or declaration: `"expected ';' after top level declarator">;`.
  **L270 CN**: 添加一条独立语句或声明：`"expected ';' after top level declarator">;`。
- **L271 EN**: Declares TableGen def record `err_invalid_token_after_declarator_suggest_equal`.
  **L271 CN**: 声明 TableGen def 记录 `err_invalid_token_after_declarator_suggest_equal`。
- **L272 EN**: Adds a standalone statement or declaration: `"invalid %0 at end of declaration; did you mean '='?">;`.
  **L272 CN**: 添加一条独立语句或声明：`"invalid %0 at end of declaration; did you mean '='?">;`。
- **L273 EN**: Declares TableGen def record `err_expected_statement`.
  **L273 CN**: 声明 TableGen def 记录 `err_expected_statement`。
- **L274 EN**: Declares TableGen def record `err_expected_lparen_after`.
  **L274 CN**: 声明 TableGen def 记录 `err_expected_lparen_after`。
- **L275 EN**: Declares TableGen def record `err_expected_rparen_after`.
  **L275 CN**: 声明 TableGen def 记录 `err_expected_rparen_after`。
- **L276 EN**: Declares TableGen def record `err_expected_punc`.
  **L276 CN**: 声明 TableGen def 记录 `err_expected_punc`。
- **L277 EN**: Declares TableGen def record `err_expected_less_after`.
  **L277 CN**: 声明 TableGen def 记录 `err_expected_less_after`。
- **L278 EN**: Declares TableGen def record `err_expected_lbrace_in_compound_literal`.
  **L278 CN**: 声明 TableGen def 记录 `err_expected_lbrace_in_compound_literal`。
- **L279 EN**: Adds a standalone statement or declaration: `"expected '{' in compound literal">;`.
  **L279 CN**: 添加一条独立语句或声明：`"expected '{' in compound literal">;`。
- **L280 EN**: Declares TableGen def record `err_expected_while`.
  **L280 CN**: 声明 TableGen def 记录 `err_expected_while`。
- **L281 EN**: Blank line separating nearby declarations or logic blocks.
  **L281 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L282 EN**: Declares TableGen def record `err_expected_semi_after_stmt`.
  **L282 CN**: 声明 TableGen def 记录 `err_expected_semi_after_stmt`。
- **L283 EN**: Declares TableGen def record `err_expected_semi_after_expr`.
  **L283 CN**: 声明 TableGen def 记录 `err_expected_semi_after_expr`。
- **L284 EN**: Declares TableGen def record `warn_attr_in_secondary_block`.
  **L284 CN**: 声明 TableGen def 记录 `warn_attr_in_secondary_block`。
- **L285 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"ISO C does not allow an attribute list to appear here">,`.
  **L285 CN**: 继续一个多行参数列表、初始化器或聚合项：`"ISO C does not allow an attribute list to appear here">,`。
- **L286 EN**: Adds a standalone statement or declaration: `InGroup<DiagGroup<"c-attribute-extension">>;`.
  **L286 CN**: 添加一条独立语句或声明：`InGroup<DiagGroup<"c-attribute-extension">>;`。
- **L287 EN**: Declares TableGen def record `err_extraneous_token_before_semi`.
  **L287 CN**: 声明 TableGen def 记录 `err_extraneous_token_before_semi`。
- **L288 EN**: Blank line separating nearby declarations or logic blocks.
  **L288 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 289-312

````tablegen
def err_expected_semi_after_method_proto : Error<
  "expected ';' after method prototype">;
def err_expected_semi_after_namespace_name : Error<
  "expected ';' after namespace name">;
def err_unexpected_namespace_attributes_alias : Error<
  "attributes cannot be specified on namespace alias">;
def err_unexpected_qualified_namespace_alias : Error<
  "namespace alias must be a single identifier">;
def err_unexpected_nested_namespace_attribute : Error<
  "attributes cannot be specified on a nested namespace definition">;
def err_inline_namespace_alias : Error<"namespace alias cannot be inline">;
def err_namespace_nonnamespace_scope : Error<
  "namespaces can only be defined in global or namespace scope">;
def ext_nested_namespace_definition : ExtWarn<
  "nested namespace definition is a C++17 extension; "
  "define each namespace separately">, InGroup<CXX17>;
def warn_cxx14_compat_nested_namespace_definition : Warning<
  "nested namespace definition is incompatible with C++ standards before C++17">,
  InGroup<CXXPre17Compat>, DefaultIgnore;
def ext_inline_nested_namespace_definition : ExtWarn<
  "inline nested namespace definition is a C++20 extension">, InGroup<CXX20>;
def warn_cxx17_compat_inline_nested_namespace_definition : Warning<
  "inline nested namespace definition is incompatible with C++ standards before"
  " C++20">, InGroup<CXXPre20Compat>, DefaultIgnore;
````
- **L289 EN**: Declares TableGen def record `err_expected_semi_after_method_proto`.
  **L289 CN**: 声明 TableGen def 记录 `err_expected_semi_after_method_proto`。
- **L290 EN**: Adds a standalone statement or declaration: `"expected ';' after method prototype">;`.
  **L290 CN**: 添加一条独立语句或声明：`"expected ';' after method prototype">;`。
- **L291 EN**: Declares TableGen def record `err_expected_semi_after_namespace_name`.
  **L291 CN**: 声明 TableGen def 记录 `err_expected_semi_after_namespace_name`。
- **L292 EN**: Adds a standalone statement or declaration: `"expected ';' after namespace name">;`.
  **L292 CN**: 添加一条独立语句或声明：`"expected ';' after namespace name">;`。
- **L293 EN**: Declares TableGen def record `err_unexpected_namespace_attributes_alias`.
  **L293 CN**: 声明 TableGen def 记录 `err_unexpected_namespace_attributes_alias`。
- **L294 EN**: Adds a standalone statement or declaration: `"attributes cannot be specified on namespace alias">;`.
  **L294 CN**: 添加一条独立语句或声明：`"attributes cannot be specified on namespace alias">;`。
- **L295 EN**: Declares TableGen def record `err_unexpected_qualified_namespace_alias`.
  **L295 CN**: 声明 TableGen def 记录 `err_unexpected_qualified_namespace_alias`。
- **L296 EN**: Adds a standalone statement or declaration: `"namespace alias must be a single identifier">;`.
  **L296 CN**: 添加一条独立语句或声明：`"namespace alias must be a single identifier">;`。
- **L297 EN**: Declares TableGen def record `err_unexpected_nested_namespace_attribute`.
  **L297 CN**: 声明 TableGen def 记录 `err_unexpected_nested_namespace_attribute`。
- **L298 EN**: Adds a standalone statement or declaration: `"attributes cannot be specified on a nested namespace definition">;`.
  **L298 CN**: 添加一条独立语句或声明：`"attributes cannot be specified on a nested namespace definition">;`。
- **L299 EN**: Declares TableGen def record `err_inline_namespace_alias`.
  **L299 CN**: 声明 TableGen def 记录 `err_inline_namespace_alias`。
- **L300 EN**: Declares TableGen def record `err_namespace_nonnamespace_scope`.
  **L300 CN**: 声明 TableGen def 记录 `err_namespace_nonnamespace_scope`。
- **L301 EN**: Adds a standalone statement or declaration: `"namespaces can only be defined in global or namespace scope">;`.
  **L301 CN**: 添加一条独立语句或声明：`"namespaces can only be defined in global or namespace scope">;`。
- **L302 EN**: Declares TableGen def record `ext_nested_namespace_definition`.
  **L302 CN**: 声明 TableGen def 记录 `ext_nested_namespace_definition`。
- **L303 EN**: Continues the surrounding expression or declaration: `"nested namespace definition is a C++17 extension; "`.
  **L303 CN**: 继续构造周围的表达式或声明：`"nested namespace definition is a C++17 extension; "`。
- **L304 EN**: Adds a standalone statement or declaration: `"define each namespace separately">, InGroup<CXX17>;`.
  **L304 CN**: 添加一条独立语句或声明：`"define each namespace separately">, InGroup<CXX17>;`。
- **L305 EN**: Declares TableGen def record `warn_cxx14_compat_nested_namespace_definition`.
  **L305 CN**: 声明 TableGen def 记录 `warn_cxx14_compat_nested_namespace_definition`。
- **L306 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"nested namespace definition is incompatible with C++ standards before C++17">,`.
  **L306 CN**: 继续一个多行参数列表、初始化器或聚合项：`"nested namespace definition is incompatible with C++ standards before C++17">,`。
- **L307 EN**: Adds a standalone statement or declaration: `InGroup<CXXPre17Compat>, DefaultIgnore;`.
  **L307 CN**: 添加一条独立语句或声明：`InGroup<CXXPre17Compat>, DefaultIgnore;`。
- **L308 EN**: Declares TableGen def record `ext_inline_nested_namespace_definition`.
  **L308 CN**: 声明 TableGen def 记录 `ext_inline_nested_namespace_definition`。
- **L309 EN**: Adds a standalone statement or declaration: `"inline nested namespace definition is a C++20 extension">, InGroup<CXX20>;`.
  **L309 CN**: 添加一条独立语句或声明：`"inline nested namespace definition is a C++20 extension">, InGroup<CXX20>;`。
- **L310 EN**: Declares TableGen def record `warn_cxx17_compat_inline_nested_namespace_definition`.
  **L310 CN**: 声明 TableGen def 记录 `warn_cxx17_compat_inline_nested_namespace_definition`。
- **L311 EN**: Continues the surrounding expression or declaration: `"inline nested namespace definition is incompatible with C++ standards before"`.
  **L311 CN**: 继续构造周围的表达式或声明：`"inline nested namespace definition is incompatible with C++ standards before"`。
- **L312 EN**: Adds a standalone statement or declaration: `" C++20">, InGroup<CXXPre20Compat>, DefaultIgnore;`.
  **L312 CN**: 添加一条独立语句或声明：`" C++20">, InGroup<CXXPre20Compat>, DefaultIgnore;`。

### Lines 313-336

````tablegen
def err_inline_nested_namespace_definition : Error<
  "nested namespace definition cannot be 'inline'">;
def err_expected_semi_after_attribute_list : Error<
  "expected ';' after attribute list">;
def err_expected_semi_after_static_assert : Error<
  "expected ';' after '%0'">;
def err_expected_semi_for : Error<"expected ';' in 'for' statement specifier">;
def err_single_decl_assign_in_for_range : Error<
  "range-based 'for' statement uses ':', not '='">;
def warn_missing_selector_name : Warning<
  "%0 used as the name of the previous parameter rather than as part "
  "of the selector">,
  InGroup<DiagGroup<"missing-selector-name">>;
def note_missing_selector_name : Note<
  "introduce a parameter name to make %0 part of the selector">;
def note_force_empty_selector_name : Note<
  "or insert whitespace before ':' to use %0 as parameter name "
  "and have an empty entry in the selector">;
def ext_c_label_followed_by_declaration : ExtWarn<
  "label followed by a declaration is a C23 extension">,
  InGroup<C23>;
def warn_c23_compat_label_followed_by_declaration : Warning<
  "label followed by a declaration is incompatible with C standards before "
  "C23">, InGroup<CPre23Compat>, DefaultIgnore;
````
- **L313 EN**: Declares TableGen def record `err_inline_nested_namespace_definition`.
  **L313 CN**: 声明 TableGen def 记录 `err_inline_nested_namespace_definition`。
- **L314 EN**: Adds a standalone statement or declaration: `"nested namespace definition cannot be 'inline'">;`.
  **L314 CN**: 添加一条独立语句或声明：`"nested namespace definition cannot be 'inline'">;`。
- **L315 EN**: Declares TableGen def record `err_expected_semi_after_attribute_list`.
  **L315 CN**: 声明 TableGen def 记录 `err_expected_semi_after_attribute_list`。
- **L316 EN**: Adds a standalone statement or declaration: `"expected ';' after attribute list">;`.
  **L316 CN**: 添加一条独立语句或声明：`"expected ';' after attribute list">;`。
- **L317 EN**: Declares TableGen def record `err_expected_semi_after_static_assert`.
  **L317 CN**: 声明 TableGen def 记录 `err_expected_semi_after_static_assert`。
- **L318 EN**: Adds a standalone statement or declaration: `"expected ';' after '%0'">;`.
  **L318 CN**: 添加一条独立语句或声明：`"expected ';' after '%0'">;`。
- **L319 EN**: Declares TableGen def record `err_expected_semi_for`.
  **L319 CN**: 声明 TableGen def 记录 `err_expected_semi_for`。
- **L320 EN**: Declares TableGen def record `err_single_decl_assign_in_for_range`.
  **L320 CN**: 声明 TableGen def 记录 `err_single_decl_assign_in_for_range`。
- **L321 EN**: Adds a standalone statement or declaration: `"range-based 'for' statement uses ':', not '='">;`.
  **L321 CN**: 添加一条独立语句或声明：`"range-based 'for' statement uses ':', not '='">;`。
- **L322 EN**: Declares TableGen def record `warn_missing_selector_name`.
  **L322 CN**: 声明 TableGen def 记录 `warn_missing_selector_name`。
- **L323 EN**: Continues the surrounding expression or declaration: `"%0 used as the name of the previous parameter rather than as part "`.
  **L323 CN**: 继续构造周围的表达式或声明：`"%0 used as the name of the previous parameter rather than as part "`。
- **L324 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"of the selector">,`.
  **L324 CN**: 继续一个多行参数列表、初始化器或聚合项：`"of the selector">,`。
- **L325 EN**: Adds a standalone statement or declaration: `InGroup<DiagGroup<"missing-selector-name">>;`.
  **L325 CN**: 添加一条独立语句或声明：`InGroup<DiagGroup<"missing-selector-name">>;`。
- **L326 EN**: Declares TableGen def record `note_missing_selector_name`.
  **L326 CN**: 声明 TableGen def 记录 `note_missing_selector_name`。
- **L327 EN**: Adds a standalone statement or declaration: `"introduce a parameter name to make %0 part of the selector">;`.
  **L327 CN**: 添加一条独立语句或声明：`"introduce a parameter name to make %0 part of the selector">;`。
- **L328 EN**: Declares TableGen def record `note_force_empty_selector_name`.
  **L328 CN**: 声明 TableGen def 记录 `note_force_empty_selector_name`。
- **L329 EN**: Continues the surrounding expression or declaration: `"or insert whitespace before ':' to use %0 as parameter name "`.
  **L329 CN**: 继续构造周围的表达式或声明：`"or insert whitespace before ':' to use %0 as parameter name "`。
- **L330 EN**: Adds a standalone statement or declaration: `"and have an empty entry in the selector">;`.
  **L330 CN**: 添加一条独立语句或声明：`"and have an empty entry in the selector">;`。
- **L331 EN**: Declares TableGen def record `ext_c_label_followed_by_declaration`.
  **L331 CN**: 声明 TableGen def 记录 `ext_c_label_followed_by_declaration`。
- **L332 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"label followed by a declaration is a C23 extension">,`.
  **L332 CN**: 继续一个多行参数列表、初始化器或聚合项：`"label followed by a declaration is a C23 extension">,`。
- **L333 EN**: Adds a standalone statement or declaration: `InGroup<C23>;`.
  **L333 CN**: 添加一条独立语句或声明：`InGroup<C23>;`。
- **L334 EN**: Declares TableGen def record `warn_c23_compat_label_followed_by_declaration`.
  **L334 CN**: 声明 TableGen def 记录 `warn_c23_compat_label_followed_by_declaration`。
- **L335 EN**: Continues the surrounding expression or declaration: `"label followed by a declaration is incompatible with C standards before "`.
  **L335 CN**: 继续构造周围的表达式或声明：`"label followed by a declaration is incompatible with C standards before "`。
- **L336 EN**: Adds a standalone statement or declaration: `"C23">, InGroup<CPre23Compat>, DefaultIgnore;`.
  **L336 CN**: 添加一条独立语句或声明：`"C23">, InGroup<CPre23Compat>, DefaultIgnore;`。

### Lines 337-360

````tablegen
def ext_c_label_end_of_compound_statement : ExtWarn<
  "label at end of compound statement is a C23 extension">,
   InGroup<C23>;
def ext_cxx_label_end_of_compound_statement : ExtWarn<
  "label at end of compound statement is a C++23 extension">,
   InGroup<CXX23>;
def warn_c23_compat_label_end_of_compound_statement : Warning<
  "label at end of compound statement is incompatible with C standards before C23">,
  InGroup<CPre23Compat>, DefaultIgnore;
def warn_cxx20_compat_label_end_of_compound_statement : Warning<
  "label at end of compound statement is incompatible with C++ standards before C++23">,
  InGroup<CXXPre23Compat>, DefaultIgnore;
def err_address_of_label_outside_fn : Error<
  "use of address-of-label extension outside of a function body">;
def err_asm_operand_wide_string_literal : Error<
  "cannot use %select{unicode|wide}0 string literal in 'asm'">;
def err_defer_ts_labeled_stmt : Error<
  "substatement of defer must not be a label">;

def err_asm_expected_string : Error<
  "expected string literal %select{or parenthesized constant expression |}0in 'asm'">;
def err_expected_selector_for_method : Error<
  "expected selector for Objective-C method">;
def err_expected_property_name : Error<"expected property name">;
````
- **L337 EN**: Declares TableGen def record `ext_c_label_end_of_compound_statement`.
  **L337 CN**: 声明 TableGen def 记录 `ext_c_label_end_of_compound_statement`。
- **L338 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"label at end of compound statement is a C23 extension">,`.
  **L338 CN**: 继续一个多行参数列表、初始化器或聚合项：`"label at end of compound statement is a C23 extension">,`。
- **L339 EN**: Adds a standalone statement or declaration: `InGroup<C23>;`.
  **L339 CN**: 添加一条独立语句或声明：`InGroup<C23>;`。
- **L340 EN**: Declares TableGen def record `ext_cxx_label_end_of_compound_statement`.
  **L340 CN**: 声明 TableGen def 记录 `ext_cxx_label_end_of_compound_statement`。
- **L341 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"label at end of compound statement is a C++23 extension">,`.
  **L341 CN**: 继续一个多行参数列表、初始化器或聚合项：`"label at end of compound statement is a C++23 extension">,`。
- **L342 EN**: Adds a standalone statement or declaration: `InGroup<CXX23>;`.
  **L342 CN**: 添加一条独立语句或声明：`InGroup<CXX23>;`。
- **L343 EN**: Declares TableGen def record `warn_c23_compat_label_end_of_compound_statement`.
  **L343 CN**: 声明 TableGen def 记录 `warn_c23_compat_label_end_of_compound_statement`。
- **L344 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"label at end of compound statement is incompatible with C standards before C23">,`.
  **L344 CN**: 继续一个多行参数列表、初始化器或聚合项：`"label at end of compound statement is incompatible with C standards before C23">,`。
- **L345 EN**: Adds a standalone statement or declaration: `InGroup<CPre23Compat>, DefaultIgnore;`.
  **L345 CN**: 添加一条独立语句或声明：`InGroup<CPre23Compat>, DefaultIgnore;`。
- **L346 EN**: Declares TableGen def record `warn_cxx20_compat_label_end_of_compound_statement`.
  **L346 CN**: 声明 TableGen def 记录 `warn_cxx20_compat_label_end_of_compound_statement`。
- **L347 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"label at end of compound statement is incompatible with C++ standards before C++23">,`.
  **L347 CN**: 继续一个多行参数列表、初始化器或聚合项：`"label at end of compound statement is incompatible with C++ standards before C++23">,`。
- **L348 EN**: Adds a standalone statement or declaration: `InGroup<CXXPre23Compat>, DefaultIgnore;`.
  **L348 CN**: 添加一条独立语句或声明：`InGroup<CXXPre23Compat>, DefaultIgnore;`。
- **L349 EN**: Declares TableGen def record `err_address_of_label_outside_fn`.
  **L349 CN**: 声明 TableGen def 记录 `err_address_of_label_outside_fn`。
- **L350 EN**: Adds a standalone statement or declaration: `"use of address-of-label extension outside of a function body">;`.
  **L350 CN**: 添加一条独立语句或声明：`"use of address-of-label extension outside of a function body">;`。
- **L351 EN**: Declares TableGen def record `err_asm_operand_wide_string_literal`.
  **L351 CN**: 声明 TableGen def 记录 `err_asm_operand_wide_string_literal`。
- **L352 EN**: Adds a standalone statement or declaration: `"cannot use %select{unicode|wide}0 string literal in 'asm'">;`.
  **L352 CN**: 添加一条独立语句或声明：`"cannot use %select{unicode|wide}0 string literal in 'asm'">;`。
- **L353 EN**: Declares TableGen def record `err_defer_ts_labeled_stmt`.
  **L353 CN**: 声明 TableGen def 记录 `err_defer_ts_labeled_stmt`。
- **L354 EN**: Adds a standalone statement or declaration: `"substatement of defer must not be a label">;`.
  **L354 CN**: 添加一条独立语句或声明：`"substatement of defer must not be a label">;`。
- **L355 EN**: Blank line separating nearby declarations or logic blocks.
  **L355 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L356 EN**: Declares TableGen def record `err_asm_expected_string`.
  **L356 CN**: 声明 TableGen def 记录 `err_asm_expected_string`。
- **L357 EN**: Adds a standalone statement or declaration: `"expected string literal %select{or parenthesized constant expression |}0in 'asm'">;`.
  **L357 CN**: 添加一条独立语句或声明：`"expected string literal %select{or parenthesized constant expression |}0in 'asm'">;`。
- **L358 EN**: Declares TableGen def record `err_expected_selector_for_method`.
  **L358 CN**: 声明 TableGen def 记录 `err_expected_selector_for_method`。
- **L359 EN**: Adds a standalone statement or declaration: `"expected selector for Objective-C method">;`.
  **L359 CN**: 添加一条独立语句或声明：`"expected selector for Objective-C method">;`。
- **L360 EN**: Declares TableGen def record `err_expected_property_name`.
  **L360 CN**: 声明 TableGen def 记录 `err_expected_property_name`。

### Lines 361-384

````tablegen

def err_unexpected_at : Error<"unexpected '@' in program">;
def err_atimport : Error<
"use of '@import' when modules are disabled">;

def warn_atimport_in_framework_header : Warning<
  "use of '@import' in framework header is discouraged, "
  "including this header requires -fmodules">,
  InGroup<FrameworkHdrAtImport>;

def err_invalid_reference_qualifier_application : Error<
  "'%0' qualifier may not be applied to a reference">;
def err_illegal_decl_reference_to_reference : Error<
  "%0 declared as a reference to a reference">;
def ext_rvalue_reference : ExtWarn<
  "rvalue references are a C++11 extension">, InGroup<CXX11>;
def warn_cxx98_compat_rvalue_reference : Warning<
  "rvalue references are incompatible with C++98">,
  InGroup<CXX98Compat>, DefaultIgnore;
def ext_ref_qualifier : ExtWarn<
  "reference qualifiers on functions are a C++11 extension">, InGroup<CXX11>;
def warn_cxx98_compat_ref_qualifier : Warning<
  "reference qualifiers on functions are incompatible with C++98">,
  InGroup<CXX98Compat>, DefaultIgnore;
````
- **L361 EN**: Blank line separating nearby declarations or logic blocks.
  **L361 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L362 EN**: Declares TableGen def record `err_unexpected_at`.
  **L362 CN**: 声明 TableGen def 记录 `err_unexpected_at`。
- **L363 EN**: Declares TableGen def record `err_atimport`.
  **L363 CN**: 声明 TableGen def 记录 `err_atimport`。
- **L364 EN**: Adds a standalone statement or declaration: `"use of '@import' when modules are disabled">;`.
  **L364 CN**: 添加一条独立语句或声明：`"use of '@import' when modules are disabled">;`。
- **L365 EN**: Blank line separating nearby declarations or logic blocks.
  **L365 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L366 EN**: Declares TableGen def record `warn_atimport_in_framework_header`.
  **L366 CN**: 声明 TableGen def 记录 `warn_atimport_in_framework_header`。
- **L367 EN**: Continues the surrounding expression or declaration: `"use of '@import' in framework header is discouraged, "`.
  **L367 CN**: 继续构造周围的表达式或声明：`"use of '@import' in framework header is discouraged, "`。
- **L368 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"including this header requires -fmodules">,`.
  **L368 CN**: 继续一个多行参数列表、初始化器或聚合项：`"including this header requires -fmodules">,`。
- **L369 EN**: Adds a standalone statement or declaration: `InGroup<FrameworkHdrAtImport>;`.
  **L369 CN**: 添加一条独立语句或声明：`InGroup<FrameworkHdrAtImport>;`。
- **L370 EN**: Blank line separating nearby declarations or logic blocks.
  **L370 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L371 EN**: Declares TableGen def record `err_invalid_reference_qualifier_application`.
  **L371 CN**: 声明 TableGen def 记录 `err_invalid_reference_qualifier_application`。
- **L372 EN**: Adds a standalone statement or declaration: `"'%0' qualifier may not be applied to a reference">;`.
  **L372 CN**: 添加一条独立语句或声明：`"'%0' qualifier may not be applied to a reference">;`。
- **L373 EN**: Declares TableGen def record `err_illegal_decl_reference_to_reference`.
  **L373 CN**: 声明 TableGen def 记录 `err_illegal_decl_reference_to_reference`。
- **L374 EN**: Adds a standalone statement or declaration: `"%0 declared as a reference to a reference">;`.
  **L374 CN**: 添加一条独立语句或声明：`"%0 declared as a reference to a reference">;`。
- **L375 EN**: Declares TableGen def record `ext_rvalue_reference`.
  **L375 CN**: 声明 TableGen def 记录 `ext_rvalue_reference`。
- **L376 EN**: Adds a standalone statement or declaration: `"rvalue references are a C++11 extension">, InGroup<CXX11>;`.
  **L376 CN**: 添加一条独立语句或声明：`"rvalue references are a C++11 extension">, InGroup<CXX11>;`。
- **L377 EN**: Declares TableGen def record `warn_cxx98_compat_rvalue_reference`.
  **L377 CN**: 声明 TableGen def 记录 `warn_cxx98_compat_rvalue_reference`。
- **L378 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"rvalue references are incompatible with C++98">,`.
  **L378 CN**: 继续一个多行参数列表、初始化器或聚合项：`"rvalue references are incompatible with C++98">,`。
- **L379 EN**: Adds a standalone statement or declaration: `InGroup<CXX98Compat>, DefaultIgnore;`.
  **L379 CN**: 添加一条独立语句或声明：`InGroup<CXX98Compat>, DefaultIgnore;`。
- **L380 EN**: Declares TableGen def record `ext_ref_qualifier`.
  **L380 CN**: 声明 TableGen def 记录 `ext_ref_qualifier`。
- **L381 EN**: Adds a standalone statement or declaration: `"reference qualifiers on functions are a C++11 extension">, InGroup<CXX11>;`.
  **L381 CN**: 添加一条独立语句或声明：`"reference qualifiers on functions are a C++11 extension">, InGroup<CXX11>;`。
- **L382 EN**: Declares TableGen def record `warn_cxx98_compat_ref_qualifier`.
  **L382 CN**: 声明 TableGen def 记录 `warn_cxx98_compat_ref_qualifier`。
- **L383 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"reference qualifiers on functions are incompatible with C++98">,`.
  **L383 CN**: 继续一个多行参数列表、初始化器或聚合项：`"reference qualifiers on functions are incompatible with C++98">,`。
- **L384 EN**: Adds a standalone statement or declaration: `InGroup<CXX98Compat>, DefaultIgnore;`.
  **L384 CN**: 添加一条独立语句或声明：`InGroup<CXX98Compat>, DefaultIgnore;`。

### Lines 385-408

````tablegen
def ext_inline_namespace : ExtWarn<
  "inline namespaces are a C++11 feature">, InGroup<CXX11InlineNamespace>;
def warn_cxx98_compat_inline_namespace : Warning<
  "inline namespaces are incompatible with C++98">,
  InGroup<CXX98Compat>, DefaultIgnore;
def ext_generalized_initializer_lists : ExtWarn<
  "generalized initializer lists are a C++11 extension">,
  InGroup<CXX11>;
def warn_cxx98_compat_generalized_initializer_lists : Warning<
  "generalized initializer lists are incompatible with C++98">,
  InGroup<CXX98Compat>, DefaultIgnore;
def err_init_list_bin_op : Error<"initializer list cannot be used on the "
  "%select{left|right}0 hand side of operator '%1'">;
def warn_cxx98_compat_trailing_return_type : Warning<
  "trailing return types are incompatible with C++98">,
  InGroup<CXX98Compat>, DefaultIgnore;
def err_requires_clause_must_appear_after_trailing_return : Error<
  "trailing return type must appear before trailing requires clause">;
def err_requires_clause_on_declarator_not_declaring_a_function : Error<
  "trailing requires clause can only be used when declaring a function">;
def err_requires_clause_inside_parens : Error<
  "trailing requires clause should be placed outside parentheses">;
def ext_auto_storage_class : ExtWarn<
  "'auto' storage class specifier is not permitted in C++11, and will not "
````
- **L385 EN**: Declares TableGen def record `ext_inline_namespace`.
  **L385 CN**: 声明 TableGen def 记录 `ext_inline_namespace`。
- **L386 EN**: Adds a standalone statement or declaration: `"inline namespaces are a C++11 feature">, InGroup<CXX11InlineNamespace>;`.
  **L386 CN**: 添加一条独立语句或声明：`"inline namespaces are a C++11 feature">, InGroup<CXX11InlineNamespace>;`。
- **L387 EN**: Declares TableGen def record `warn_cxx98_compat_inline_namespace`.
  **L387 CN**: 声明 TableGen def 记录 `warn_cxx98_compat_inline_namespace`。
- **L388 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"inline namespaces are incompatible with C++98">,`.
  **L388 CN**: 继续一个多行参数列表、初始化器或聚合项：`"inline namespaces are incompatible with C++98">,`。
- **L389 EN**: Adds a standalone statement or declaration: `InGroup<CXX98Compat>, DefaultIgnore;`.
  **L389 CN**: 添加一条独立语句或声明：`InGroup<CXX98Compat>, DefaultIgnore;`。
- **L390 EN**: Declares TableGen def record `ext_generalized_initializer_lists`.
  **L390 CN**: 声明 TableGen def 记录 `ext_generalized_initializer_lists`。
- **L391 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"generalized initializer lists are a C++11 extension">,`.
  **L391 CN**: 继续一个多行参数列表、初始化器或聚合项：`"generalized initializer lists are a C++11 extension">,`。
- **L392 EN**: Adds a standalone statement or declaration: `InGroup<CXX11>;`.
  **L392 CN**: 添加一条独立语句或声明：`InGroup<CXX11>;`。
- **L393 EN**: Declares TableGen def record `warn_cxx98_compat_generalized_initializer_lists`.
  **L393 CN**: 声明 TableGen def 记录 `warn_cxx98_compat_generalized_initializer_lists`。
- **L394 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"generalized initializer lists are incompatible with C++98">,`.
  **L394 CN**: 继续一个多行参数列表、初始化器或聚合项：`"generalized initializer lists are incompatible with C++98">,`。
- **L395 EN**: Adds a standalone statement or declaration: `InGroup<CXX98Compat>, DefaultIgnore;`.
  **L395 CN**: 添加一条独立语句或声明：`InGroup<CXX98Compat>, DefaultIgnore;`。
- **L396 EN**: Declares TableGen def record `err_init_list_bin_op`.
  **L396 CN**: 声明 TableGen def 记录 `err_init_list_bin_op`。
- **L397 EN**: Adds a standalone statement or declaration: `"%select{left|right}0 hand side of operator '%1'">;`.
  **L397 CN**: 添加一条独立语句或声明：`"%select{left|right}0 hand side of operator '%1'">;`。
- **L398 EN**: Declares TableGen def record `warn_cxx98_compat_trailing_return_type`.
  **L398 CN**: 声明 TableGen def 记录 `warn_cxx98_compat_trailing_return_type`。
- **L399 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"trailing return types are incompatible with C++98">,`.
  **L399 CN**: 继续一个多行参数列表、初始化器或聚合项：`"trailing return types are incompatible with C++98">,`。
- **L400 EN**: Adds a standalone statement or declaration: `InGroup<CXX98Compat>, DefaultIgnore;`.
  **L400 CN**: 添加一条独立语句或声明：`InGroup<CXX98Compat>, DefaultIgnore;`。
- **L401 EN**: Declares TableGen def record `err_requires_clause_must_appear_after_trailing_return`.
  **L401 CN**: 声明 TableGen def 记录 `err_requires_clause_must_appear_after_trailing_return`。
- **L402 EN**: Adds a standalone statement or declaration: `"trailing return type must appear before trailing requires clause">;`.
  **L402 CN**: 添加一条独立语句或声明：`"trailing return type must appear before trailing requires clause">;`。
- **L403 EN**: Declares TableGen def record `err_requires_clause_on_declarator_not_declaring_a_function`.
  **L403 CN**: 声明 TableGen def 记录 `err_requires_clause_on_declarator_not_declaring_a_function`。
- **L404 EN**: Adds a standalone statement or declaration: `"trailing requires clause can only be used when declaring a function">;`.
  **L404 CN**: 添加一条独立语句或声明：`"trailing requires clause can only be used when declaring a function">;`。
- **L405 EN**: Declares TableGen def record `err_requires_clause_inside_parens`.
  **L405 CN**: 声明 TableGen def 记录 `err_requires_clause_inside_parens`。
- **L406 EN**: Adds a standalone statement or declaration: `"trailing requires clause should be placed outside parentheses">;`.
  **L406 CN**: 添加一条独立语句或声明：`"trailing requires clause should be placed outside parentheses">;`。
- **L407 EN**: Declares TableGen def record `ext_auto_storage_class`.
  **L407 CN**: 声明 TableGen def 记录 `ext_auto_storage_class`。
- **L408 EN**: Continues the surrounding expression or declaration: `"'auto' storage class specifier is not permitted in C++11, and will not "`.
  **L408 CN**: 继续构造周围的表达式或声明：`"'auto' storage class specifier is not permitted in C++11, and will not "`。

### Lines 409-432

````tablegen
  "be supported in future releases">, InGroup<DiagGroup<"auto-storage-class">>;
def ext_decltype_auto_type_specifier : ExtWarn<
  "'decltype(auto)' type specifier is a C++14 extension">, InGroup<CXX14>;
def warn_cxx11_compat_decltype_auto_type_specifier : Warning<
  "'decltype(auto)' type specifier is incompatible with C++ standards before "
  "C++14">, InGroup<CXXPre14Compat>, DefaultIgnore;
def ext_auto_type : Extension<
  "'__auto_type' is a GNU extension">,
  InGroup<GNUAutoType>;
def ext_for_range : ExtWarn<
  "range-based for loop is a C++11 extension">, InGroup<CXX11>;
def warn_cxx98_compat_for_range : Warning<
  "range-based for loop is incompatible with C++98">,
  InGroup<CXX98Compat>, DefaultIgnore;
def err_for_range_identifier : Error<
  "range-based for loop requires type for loop variable">;
def err_for_range_expected_decl : Error<
  "for range declaration must declare a variable">;
def err_argument_required_after_attribute : Error<
  "argument required after attribute">;
def err_missing_param : Error<"expected parameter declarator">;
def err_function_scope_depth_exceeded : Error<
  "function scope depth exceeded maximum of %0">, DefaultFatal;
def err_missing_comma_before_ellipsis : Error<
````
- **L409 EN**: Adds a standalone statement or declaration: `"be supported in future releases">, InGroup<DiagGroup<"auto-storage-class">>;`.
  **L409 CN**: 添加一条独立语句或声明：`"be supported in future releases">, InGroup<DiagGroup<"auto-storage-class">>;`。
- **L410 EN**: Declares TableGen def record `ext_decltype_auto_type_specifier`.
  **L410 CN**: 声明 TableGen def 记录 `ext_decltype_auto_type_specifier`。
- **L411 EN**: Executes a call or declaration centered on `"'decltype`.
  **L411 CN**: 执行以 `"'decltype` 为核心的调用或声明。
- **L412 EN**: Declares TableGen def record `warn_cxx11_compat_decltype_auto_type_specifier`.
  **L412 CN**: 声明 TableGen def 记录 `warn_cxx11_compat_decltype_auto_type_specifier`。
- **L413 EN**: Continues the surrounding expression or declaration: `"'decltype(auto)' type specifier is incompatible with C++ standards before "`.
  **L413 CN**: 继续构造周围的表达式或声明：`"'decltype(auto)' type specifier is incompatible with C++ standards before "`。
- **L414 EN**: Adds a standalone statement or declaration: `"C++14">, InGroup<CXXPre14Compat>, DefaultIgnore;`.
  **L414 CN**: 添加一条独立语句或声明：`"C++14">, InGroup<CXXPre14Compat>, DefaultIgnore;`。
- **L415 EN**: Declares TableGen def record `ext_auto_type`.
  **L415 CN**: 声明 TableGen def 记录 `ext_auto_type`。
- **L416 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'__auto_type' is a GNU extension">,`.
  **L416 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'__auto_type' is a GNU extension">,`。
- **L417 EN**: Adds a standalone statement or declaration: `InGroup<GNUAutoType>;`.
  **L417 CN**: 添加一条独立语句或声明：`InGroup<GNUAutoType>;`。
- **L418 EN**: Declares TableGen def record `ext_for_range`.
  **L418 CN**: 声明 TableGen def 记录 `ext_for_range`。
- **L419 EN**: Adds a standalone statement or declaration: `"range-based for loop is a C++11 extension">, InGroup<CXX11>;`.
  **L419 CN**: 添加一条独立语句或声明：`"range-based for loop is a C++11 extension">, InGroup<CXX11>;`。
- **L420 EN**: Declares TableGen def record `warn_cxx98_compat_for_range`.
  **L420 CN**: 声明 TableGen def 记录 `warn_cxx98_compat_for_range`。
- **L421 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"range-based for loop is incompatible with C++98">,`.
  **L421 CN**: 继续一个多行参数列表、初始化器或聚合项：`"range-based for loop is incompatible with C++98">,`。
- **L422 EN**: Adds a standalone statement or declaration: `InGroup<CXX98Compat>, DefaultIgnore;`.
  **L422 CN**: 添加一条独立语句或声明：`InGroup<CXX98Compat>, DefaultIgnore;`。
- **L423 EN**: Declares TableGen def record `err_for_range_identifier`.
  **L423 CN**: 声明 TableGen def 记录 `err_for_range_identifier`。
- **L424 EN**: Adds a standalone statement or declaration: `"range-based for loop requires type for loop variable">;`.
  **L424 CN**: 添加一条独立语句或声明：`"range-based for loop requires type for loop variable">;`。
- **L425 EN**: Declares TableGen def record `err_for_range_expected_decl`.
  **L425 CN**: 声明 TableGen def 记录 `err_for_range_expected_decl`。
- **L426 EN**: Adds a standalone statement or declaration: `"for range declaration must declare a variable">;`.
  **L426 CN**: 添加一条独立语句或声明：`"for range declaration must declare a variable">;`。
- **L427 EN**: Declares TableGen def record `err_argument_required_after_attribute`.
  **L427 CN**: 声明 TableGen def 记录 `err_argument_required_after_attribute`。
- **L428 EN**: Adds a standalone statement or declaration: `"argument required after attribute">;`.
  **L428 CN**: 添加一条独立语句或声明：`"argument required after attribute">;`。
- **L429 EN**: Declares TableGen def record `err_missing_param`.
  **L429 CN**: 声明 TableGen def 记录 `err_missing_param`。
- **L430 EN**: Declares TableGen def record `err_function_scope_depth_exceeded`.
  **L430 CN**: 声明 TableGen def 记录 `err_function_scope_depth_exceeded`。
- **L431 EN**: Adds a standalone statement or declaration: `"function scope depth exceeded maximum of %0">, DefaultFatal;`.
  **L431 CN**: 添加一条独立语句或声明：`"function scope depth exceeded maximum of %0">, DefaultFatal;`。
- **L432 EN**: Declares TableGen def record `err_missing_comma_before_ellipsis`.
  **L432 CN**: 声明 TableGen def 记录 `err_missing_comma_before_ellipsis`。

### Lines 433-456

````tablegen
  "C requires a comma prior to the ellipsis in a variadic function type">;
def warn_deprecated_missing_comma_before_ellipsis : Warning<
  "declaration of a variadic function without a comma before '...' is deprecated">,
  InGroup<DeprecatedMissingCommaVariadicParam>;
def err_unexpected_typedef_ident : Error<
  "unexpected type name %0: expected identifier">;
def warn_cxx98_compat_decltype : Warning<
  "'decltype' type specifier is incompatible with C++98">,
  InGroup<CXX98Compat>, DefaultIgnore;
def err_unexpected_scope_on_base_decltype : Error<
  "unexpected namespace scope prior to decltype">;
def err_expected_class_name : Error<"expected class name">;
def err_expected_class_name_not_template :
  Error<"'typename' is redundant; base classes are implicitly types">;
def err_unspecified_vla_size_with_static : Error<
  "'static' may not be used with an unspecified variable length array size">;
def err_unspecified_size_with_static : Error<
  "'static' may not be used without an array size">;
def err_expected_parentheses_around_typename : Error<
  "expected parentheses around type name in %0 expression">;

def err_expected_case_before_expression: Error<
  "expected 'case' keyword before expression">;

````
- **L433 EN**: Adds a standalone statement or declaration: `"C requires a comma prior to the ellipsis in a variadic function type">;`.
  **L433 CN**: 添加一条独立语句或声明：`"C requires a comma prior to the ellipsis in a variadic function type">;`。
- **L434 EN**: Declares TableGen def record `warn_deprecated_missing_comma_before_ellipsis`.
  **L434 CN**: 声明 TableGen def 记录 `warn_deprecated_missing_comma_before_ellipsis`。
- **L435 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"declaration of a variadic function without a comma before '...' is deprecated">,`.
  **L435 CN**: 继续一个多行参数列表、初始化器或聚合项：`"declaration of a variadic function without a comma before '...' is deprecated">,`。
- **L436 EN**: Adds a standalone statement or declaration: `InGroup<DeprecatedMissingCommaVariadicParam>;`.
  **L436 CN**: 添加一条独立语句或声明：`InGroup<DeprecatedMissingCommaVariadicParam>;`。
- **L437 EN**: Declares TableGen def record `err_unexpected_typedef_ident`.
  **L437 CN**: 声明 TableGen def 记录 `err_unexpected_typedef_ident`。
- **L438 EN**: Adds a standalone statement or declaration: `"unexpected type name %0: expected identifier">;`.
  **L438 CN**: 添加一条独立语句或声明：`"unexpected type name %0: expected identifier">;`。
- **L439 EN**: Declares TableGen def record `warn_cxx98_compat_decltype`.
  **L439 CN**: 声明 TableGen def 记录 `warn_cxx98_compat_decltype`。
- **L440 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'decltype' type specifier is incompatible with C++98">,`.
  **L440 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'decltype' type specifier is incompatible with C++98">,`。
- **L441 EN**: Adds a standalone statement or declaration: `InGroup<CXX98Compat>, DefaultIgnore;`.
  **L441 CN**: 添加一条独立语句或声明：`InGroup<CXX98Compat>, DefaultIgnore;`。
- **L442 EN**: Declares TableGen def record `err_unexpected_scope_on_base_decltype`.
  **L442 CN**: 声明 TableGen def 记录 `err_unexpected_scope_on_base_decltype`。
- **L443 EN**: Adds a standalone statement or declaration: `"unexpected namespace scope prior to decltype">;`.
  **L443 CN**: 添加一条独立语句或声明：`"unexpected namespace scope prior to decltype">;`。
- **L444 EN**: Declares TableGen def record `err_expected_class_name`.
  **L444 CN**: 声明 TableGen def 记录 `err_expected_class_name`。
- **L445 EN**: Declares TableGen def record `err_expected_class_name_not_template`.
  **L445 CN**: 声明 TableGen def 记录 `err_expected_class_name_not_template`。
- **L446 EN**: Adds a standalone statement or declaration: `Error<"'typename' is redundant; base classes are implicitly types">;`.
  **L446 CN**: 添加一条独立语句或声明：`Error<"'typename' is redundant; base classes are implicitly types">;`。
- **L447 EN**: Declares TableGen def record `err_unspecified_vla_size_with_static`.
  **L447 CN**: 声明 TableGen def 记录 `err_unspecified_vla_size_with_static`。
- **L448 EN**: Adds a standalone statement or declaration: `"'static' may not be used with an unspecified variable length array size">;`.
  **L448 CN**: 添加一条独立语句或声明：`"'static' may not be used with an unspecified variable length array size">;`。
- **L449 EN**: Declares TableGen def record `err_unspecified_size_with_static`.
  **L449 CN**: 声明 TableGen def 记录 `err_unspecified_size_with_static`。
- **L450 EN**: Adds a standalone statement or declaration: `"'static' may not be used without an array size">;`.
  **L450 CN**: 添加一条独立语句或声明：`"'static' may not be used without an array size">;`。
- **L451 EN**: Declares TableGen def record `err_expected_parentheses_around_typename`.
  **L451 CN**: 声明 TableGen def 记录 `err_expected_parentheses_around_typename`。
- **L452 EN**: Adds a standalone statement or declaration: `"expected parentheses around type name in %0 expression">;`.
  **L452 CN**: 添加一条独立语句或声明：`"expected parentheses around type name in %0 expression">;`。
- **L453 EN**: Blank line separating nearby declarations or logic blocks.
  **L453 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L454 EN**: Declares TableGen def record `err_expected_case_before_expression`.
  **L454 CN**: 声明 TableGen def 记录 `err_expected_case_before_expression`。
- **L455 EN**: Adds a standalone statement or declaration: `"expected 'case' keyword before expression">;`.
  **L455 CN**: 添加一条独立语句或声明：`"expected 'case' keyword before expression">;`。
- **L456 EN**: Blank line separating nearby declarations or logic blocks.
  **L456 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 457-480

````tablegen
def ext_warn_gnu_final : ExtWarn<
  "__final is a GNU extension, consider using C++11 final">,
  InGroup<GccCompat>;

// Declarations.
def err_typename_requires_specqual : Error<
  "type name requires a specifier or qualifier">;
def err_typename_invalid_storageclass : Error<
  "type name does not allow storage class to be specified">;
def err_typename_invalid_functionspec : Error<
  "type name does not allow function specifier to be specified">;
def err_typename_invalid_constexpr : Error<
  "type name does not allow %sub{select_constexpr_spec_kind}0 specifier "
  "to be specified">;
def err_typename_identifiers_only : Error<
  "typename is allowed for identifiers only">;

def err_friend_invalid_in_context : Error<
  "'friend' used outside of class">;
def err_templated_using_directive_declaration : Error<
  "cannot template a using %select{directive|declaration}0">;
def err_unexpected_colon_in_nested_name_spec : Error<
  "unexpected ':' in nested name specifier; did you mean '::'?">;
def err_unexpected_token_in_nested_name_spec : Error<
````
- **L457 EN**: Declares TableGen def record `ext_warn_gnu_final`.
  **L457 CN**: 声明 TableGen def 记录 `ext_warn_gnu_final`。
- **L458 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"__final is a GNU extension, consider using C++11 final">,`.
  **L458 CN**: 继续一个多行参数列表、初始化器或聚合项：`"__final is a GNU extension, consider using C++11 final">,`。
- **L459 EN**: Adds a standalone statement or declaration: `InGroup<GccCompat>;`.
  **L459 CN**: 添加一条独立语句或声明：`InGroup<GccCompat>;`。
- **L460 EN**: Blank line separating nearby declarations or logic blocks.
  **L460 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L461 EN**: Comment explains nearby logic, constraints, or intent: `Declarations.`.
  **L461 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Declarations.`。
- **L462 EN**: Declares TableGen def record `err_typename_requires_specqual`.
  **L462 CN**: 声明 TableGen def 记录 `err_typename_requires_specqual`。
- **L463 EN**: Adds a standalone statement or declaration: `"type name requires a specifier or qualifier">;`.
  **L463 CN**: 添加一条独立语句或声明：`"type name requires a specifier or qualifier">;`。
- **L464 EN**: Declares TableGen def record `err_typename_invalid_storageclass`.
  **L464 CN**: 声明 TableGen def 记录 `err_typename_invalid_storageclass`。
- **L465 EN**: Adds a standalone statement or declaration: `"type name does not allow storage class to be specified">;`.
  **L465 CN**: 添加一条独立语句或声明：`"type name does not allow storage class to be specified">;`。
- **L466 EN**: Declares TableGen def record `err_typename_invalid_functionspec`.
  **L466 CN**: 声明 TableGen def 记录 `err_typename_invalid_functionspec`。
- **L467 EN**: Adds a standalone statement or declaration: `"type name does not allow function specifier to be specified">;`.
  **L467 CN**: 添加一条独立语句或声明：`"type name does not allow function specifier to be specified">;`。
- **L468 EN**: Declares TableGen def record `err_typename_invalid_constexpr`.
  **L468 CN**: 声明 TableGen def 记录 `err_typename_invalid_constexpr`。
- **L469 EN**: Continues the surrounding expression or declaration: `"type name does not allow %sub{select_constexpr_spec_kind}0 specifier "`.
  **L469 CN**: 继续构造周围的表达式或声明：`"type name does not allow %sub{select_constexpr_spec_kind}0 specifier "`。
- **L470 EN**: Adds a standalone statement or declaration: `"to be specified">;`.
  **L470 CN**: 添加一条独立语句或声明：`"to be specified">;`。
- **L471 EN**: Declares TableGen def record `err_typename_identifiers_only`.
  **L471 CN**: 声明 TableGen def 记录 `err_typename_identifiers_only`。
- **L472 EN**: Adds a standalone statement or declaration: `"typename is allowed for identifiers only">;`.
  **L472 CN**: 添加一条独立语句或声明：`"typename is allowed for identifiers only">;`。
- **L473 EN**: Blank line separating nearby declarations or logic blocks.
  **L473 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L474 EN**: Declares TableGen def record `err_friend_invalid_in_context`.
  **L474 CN**: 声明 TableGen def 记录 `err_friend_invalid_in_context`。
- **L475 EN**: Adds a standalone statement or declaration: `"'friend' used outside of class">;`.
  **L475 CN**: 添加一条独立语句或声明：`"'friend' used outside of class">;`。
- **L476 EN**: Declares TableGen def record `err_templated_using_directive_declaration`.
  **L476 CN**: 声明 TableGen def 记录 `err_templated_using_directive_declaration`。
- **L477 EN**: Adds a standalone statement or declaration: `"cannot template a using %select{directive|declaration}0">;`.
  **L477 CN**: 添加一条独立语句或声明：`"cannot template a using %select{directive|declaration}0">;`。
- **L478 EN**: Declares TableGen def record `err_unexpected_colon_in_nested_name_spec`.
  **L478 CN**: 声明 TableGen def 记录 `err_unexpected_colon_in_nested_name_spec`。
- **L479 EN**: Adds a standalone statement or declaration: `"unexpected ':' in nested name specifier; did you mean '::'?">;`.
  **L479 CN**: 添加一条独立语句或声明：`"unexpected ':' in nested name specifier; did you mean '::'?">;`。
- **L480 EN**: Declares TableGen def record `err_unexpected_token_in_nested_name_spec`.
  **L480 CN**: 声明 TableGen def 记录 `err_unexpected_token_in_nested_name_spec`。

### Lines 481-504

````tablegen
  "'%0' cannot be a part of nested name specifier; did you mean ':'?">;
def err_bool_redeclaration : Error<
  "redeclaration of C++ built-in type 'bool'">;
def warn_cxx98_compat_static_assert : Warning<
  "'static_assert' declarations are incompatible with C++98">,
  InGroup<CXX98Compat>, DefaultIgnore;
def ext_cxx_static_assert_no_message : ExtWarn<
  "'static_assert' with no message is a C++17 extension">, InGroup<CXX17>;
def ext_c_static_assert_no_message : ExtWarn<
  "'_Static_assert' with no message is a C23 extension">, InGroup<C23>;
def warn_cxx14_compat_static_assert_no_message : Warning<
  "'static_assert' with no message is incompatible with C++ standards before "
  "C++17">,
  DefaultIgnore, InGroup<CXXPre17Compat>;
def warn_c17_compat_static_assert_no_message : Warning<
  "'_Static_assert' with no message is incompatible with C standards before "
  "C23">,
  DefaultIgnore, InGroup<CPre23Compat>;
def ext_cxx_static_assert_user_generated_message : ExtWarn<
  "'static_assert' with a user-generated message is a C++26 extension">,
  InGroup<CXX26>;
def warn_cxx20_compat_static_assert_user_generated_message : Warning<
  "'static_assert' with a user-generated message is incompatible with "
  "C++ standards before C++26">, DefaultIgnore, InGroup<CXXPre26Compat>;
````
- **L481 EN**: Adds a standalone statement or declaration: `"'%0' cannot be a part of nested name specifier; did you mean ':'?">;`.
  **L481 CN**: 添加一条独立语句或声明：`"'%0' cannot be a part of nested name specifier; did you mean ':'?">;`。
- **L482 EN**: Declares TableGen def record `err_bool_redeclaration`.
  **L482 CN**: 声明 TableGen def 记录 `err_bool_redeclaration`。
- **L483 EN**: Adds a standalone statement or declaration: `"redeclaration of C++ built-in type 'bool'">;`.
  **L483 CN**: 添加一条独立语句或声明：`"redeclaration of C++ built-in type 'bool'">;`。
- **L484 EN**: Declares TableGen def record `warn_cxx98_compat_static_assert`.
  **L484 CN**: 声明 TableGen def 记录 `warn_cxx98_compat_static_assert`。
- **L485 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'static_assert' declarations are incompatible with C++98">,`.
  **L485 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'static_assert' declarations are incompatible with C++98">,`。
- **L486 EN**: Adds a standalone statement or declaration: `InGroup<CXX98Compat>, DefaultIgnore;`.
  **L486 CN**: 添加一条独立语句或声明：`InGroup<CXX98Compat>, DefaultIgnore;`。
- **L487 EN**: Declares TableGen def record `ext_cxx_static_assert_no_message`.
  **L487 CN**: 声明 TableGen def 记录 `ext_cxx_static_assert_no_message`。
- **L488 EN**: Adds a standalone statement or declaration: `"'static_assert' with no message is a C++17 extension">, InGroup<CXX17>;`.
  **L488 CN**: 添加一条独立语句或声明：`"'static_assert' with no message is a C++17 extension">, InGroup<CXX17>;`。
- **L489 EN**: Declares TableGen def record `ext_c_static_assert_no_message`.
  **L489 CN**: 声明 TableGen def 记录 `ext_c_static_assert_no_message`。
- **L490 EN**: Adds a standalone statement or declaration: `"'_Static_assert' with no message is a C23 extension">, InGroup<C23>;`.
  **L490 CN**: 添加一条独立语句或声明：`"'_Static_assert' with no message is a C23 extension">, InGroup<C23>;`。
- **L491 EN**: Declares TableGen def record `warn_cxx14_compat_static_assert_no_message`.
  **L491 CN**: 声明 TableGen def 记录 `warn_cxx14_compat_static_assert_no_message`。
- **L492 EN**: Continues the surrounding expression or declaration: `"'static_assert' with no message is incompatible with C++ standards before "`.
  **L492 CN**: 继续构造周围的表达式或声明：`"'static_assert' with no message is incompatible with C++ standards before "`。
- **L493 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"C++17">,`.
  **L493 CN**: 继续一个多行参数列表、初始化器或聚合项：`"C++17">,`。
- **L494 EN**: Adds a standalone statement or declaration: `DefaultIgnore, InGroup<CXXPre17Compat>;`.
  **L494 CN**: 添加一条独立语句或声明：`DefaultIgnore, InGroup<CXXPre17Compat>;`。
- **L495 EN**: Declares TableGen def record `warn_c17_compat_static_assert_no_message`.
  **L495 CN**: 声明 TableGen def 记录 `warn_c17_compat_static_assert_no_message`。
- **L496 EN**: Continues the surrounding expression or declaration: `"'_Static_assert' with no message is incompatible with C standards before "`.
  **L496 CN**: 继续构造周围的表达式或声明：`"'_Static_assert' with no message is incompatible with C standards before "`。
- **L497 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"C23">,`.
  **L497 CN**: 继续一个多行参数列表、初始化器或聚合项：`"C23">,`。
- **L498 EN**: Adds a standalone statement or declaration: `DefaultIgnore, InGroup<CPre23Compat>;`.
  **L498 CN**: 添加一条独立语句或声明：`DefaultIgnore, InGroup<CPre23Compat>;`。
- **L499 EN**: Declares TableGen def record `ext_cxx_static_assert_user_generated_message`.
  **L499 CN**: 声明 TableGen def 记录 `ext_cxx_static_assert_user_generated_message`。
- **L500 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'static_assert' with a user-generated message is a C++26 extension">,`.
  **L500 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'static_assert' with a user-generated message is a C++26 extension">,`。
- **L501 EN**: Adds a standalone statement or declaration: `InGroup<CXX26>;`.
  **L501 CN**: 添加一条独立语句或声明：`InGroup<CXX26>;`。
- **L502 EN**: Declares TableGen def record `warn_cxx20_compat_static_assert_user_generated_message`.
  **L502 CN**: 声明 TableGen def 记录 `warn_cxx20_compat_static_assert_user_generated_message`。
- **L503 EN**: Continues the surrounding expression or declaration: `"'static_assert' with a user-generated message is incompatible with "`.
  **L503 CN**: 继续构造周围的表达式或声明：`"'static_assert' with a user-generated message is incompatible with "`。
- **L504 EN**: Adds a standalone statement or declaration: `"C++ standards before C++26">, DefaultIgnore, InGroup<CXXPre26Compat>;`.
  **L504 CN**: 添加一条独立语句或声明：`"C++ standards before C++26">, DefaultIgnore, InGroup<CXXPre26Compat>;`。

### Lines 505-528

````tablegen
def err_function_definition_not_allowed : Error<
  "function definition is not allowed here">;
def err_expected_end_of_enumerator : Error<
  "expected '= constant-expression' or end of enumerator definition">;
def err_expected_coloncolon_after_super : Error<
  "expected '::' after '__super'">;

def ext_decomp_decl_empty : ExtWarn<
  "ISO C++17 does not allow a structured binding group to be empty">,
  InGroup<DiagGroup<"empty-decomposition">>;

def err_function_parameter_limit_exceeded : Error<
  "too many function parameters; subsequent parameters will be ignored">;

// C++26 structured bindings
def ext_decl_attrs_on_binding : ExtWarn<
  "an attribute specifier sequence attached to a structured binding declaration "
  "is a C++2c extension">, InGroup<CXX26>;
def warn_cxx23_compat_decl_attrs_on_binding : Warning<
  "an attribute specifier sequence attached to a structured binding declaration "
  "is incompatible with C++ standards before C++2c">,
  InGroup<CXXPre26Compat>, DefaultIgnore;

/// Objective-C parser diagnostics
````
- **L505 EN**: Declares TableGen def record `err_function_definition_not_allowed`.
  **L505 CN**: 声明 TableGen def 记录 `err_function_definition_not_allowed`。
- **L506 EN**: Adds a standalone statement or declaration: `"function definition is not allowed here">;`.
  **L506 CN**: 添加一条独立语句或声明：`"function definition is not allowed here">;`。
- **L507 EN**: Declares TableGen def record `err_expected_end_of_enumerator`.
  **L507 CN**: 声明 TableGen def 记录 `err_expected_end_of_enumerator`。
- **L508 EN**: Adds a standalone statement or declaration: `"expected '= constant-expression' or end of enumerator definition">;`.
  **L508 CN**: 添加一条独立语句或声明：`"expected '= constant-expression' or end of enumerator definition">;`。
- **L509 EN**: Declares TableGen def record `err_expected_coloncolon_after_super`.
  **L509 CN**: 声明 TableGen def 记录 `err_expected_coloncolon_after_super`。
- **L510 EN**: Adds a standalone statement or declaration: `"expected '::' after '__super'">;`.
  **L510 CN**: 添加一条独立语句或声明：`"expected '::' after '__super'">;`。
- **L511 EN**: Blank line separating nearby declarations or logic blocks.
  **L511 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L512 EN**: Declares TableGen def record `ext_decomp_decl_empty`.
  **L512 CN**: 声明 TableGen def 记录 `ext_decomp_decl_empty`。
- **L513 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"ISO C++17 does not allow a structured binding group to be empty">,`.
  **L513 CN**: 继续一个多行参数列表、初始化器或聚合项：`"ISO C++17 does not allow a structured binding group to be empty">,`。
- **L514 EN**: Adds a standalone statement or declaration: `InGroup<DiagGroup<"empty-decomposition">>;`.
  **L514 CN**: 添加一条独立语句或声明：`InGroup<DiagGroup<"empty-decomposition">>;`。
- **L515 EN**: Blank line separating nearby declarations or logic blocks.
  **L515 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L516 EN**: Declares TableGen def record `err_function_parameter_limit_exceeded`.
  **L516 CN**: 声明 TableGen def 记录 `err_function_parameter_limit_exceeded`。
- **L517 EN**: Adds a standalone statement or declaration: `"too many function parameters; subsequent parameters will be ignored">;`.
  **L517 CN**: 添加一条独立语句或声明：`"too many function parameters; subsequent parameters will be ignored">;`。
- **L518 EN**: Blank line separating nearby declarations or logic blocks.
  **L518 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L519 EN**: Comment explains nearby logic, constraints, or intent: `C++26 structured bindings`.
  **L519 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C++26 structured bindings`。
- **L520 EN**: Declares TableGen def record `ext_decl_attrs_on_binding`.
  **L520 CN**: 声明 TableGen def 记录 `ext_decl_attrs_on_binding`。
- **L521 EN**: Continues the surrounding expression or declaration: `"an attribute specifier sequence attached to a structured binding declaration "`.
  **L521 CN**: 继续构造周围的表达式或声明：`"an attribute specifier sequence attached to a structured binding declaration "`。
- **L522 EN**: Adds a standalone statement or declaration: `"is a C++2c extension">, InGroup<CXX26>;`.
  **L522 CN**: 添加一条独立语句或声明：`"is a C++2c extension">, InGroup<CXX26>;`。
- **L523 EN**: Declares TableGen def record `warn_cxx23_compat_decl_attrs_on_binding`.
  **L523 CN**: 声明 TableGen def 记录 `warn_cxx23_compat_decl_attrs_on_binding`。
- **L524 EN**: Continues the surrounding expression or declaration: `"an attribute specifier sequence attached to a structured binding declaration "`.
  **L524 CN**: 继续构造周围的表达式或声明：`"an attribute specifier sequence attached to a structured binding declaration "`。
- **L525 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"is incompatible with C++ standards before C++2c">,`.
  **L525 CN**: 继续一个多行参数列表、初始化器或聚合项：`"is incompatible with C++ standards before C++2c">,`。
- **L526 EN**: Adds a standalone statement or declaration: `InGroup<CXXPre26Compat>, DefaultIgnore;`.
  **L526 CN**: 添加一条独立语句或声明：`InGroup<CXXPre26Compat>, DefaultIgnore;`。
- **L527 EN**: Blank line separating nearby declarations or logic blocks.
  **L527 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L528 EN**: Comment explains nearby logic, constraints, or intent: `Objective-C parser diagnostics`.
  **L528 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Objective-C parser diagnostics`。

### Lines 529-552

````tablegen
def err_expected_minus_or_plus : Error<
  "method type specifier must start with '-' or '+'">;
def err_objc_missing_end : Error<"missing '@end'">;
def note_objc_container_start : Note<
  "%select{class|protocol|category|class extension|implementation"
  "|category implementation}0 started here">;
def warn_objc_protocol_qualifier_missing_id : Warning<
  "protocol has no object type specified; defaults to qualified 'id'">;
def err_objc_unknown_at : Error<"expected an Objective-C directive after '@'">;
def err_illegal_super_cast : Error<
  "cannot cast 'super' (it isn't an expression)">;
def err_nsnumber_nonliteral_unary : Error<
  "@%0 must be followed by a number to form an NSNumber object">;
def warn_cstyle_param : Warning<
  "use of C-style parameters in Objective-C method declarations"
  " is deprecated">, InGroup<DeprecatedDeclarations>;

let CategoryName = "ARC Parse Issue" in {
def err_arc_bridge_retain : Error<
  "unknown cast annotation __bridge_retain; did you mean __bridge_retained?">;
// To be default mapped to an error later.
def warn_arc_bridge_cast_nonarc : Warning<
  "'%0' casts have no effect when not using ARC">,
  InGroup<DiagGroup<"arc-bridge-casts-disallowed-in-nonarc">>;
````
- **L529 EN**: Declares TableGen def record `err_expected_minus_or_plus`.
  **L529 CN**: 声明 TableGen def 记录 `err_expected_minus_or_plus`。
- **L530 EN**: Adds a standalone statement or declaration: `"method type specifier must start with '-' or '+'">;`.
  **L530 CN**: 添加一条独立语句或声明：`"method type specifier must start with '-' or '+'">;`。
- **L531 EN**: Declares TableGen def record `err_objc_missing_end`.
  **L531 CN**: 声明 TableGen def 记录 `err_objc_missing_end`。
- **L532 EN**: Declares TableGen def record `note_objc_container_start`.
  **L532 CN**: 声明 TableGen def 记录 `note_objc_container_start`。
- **L533 EN**: Continues the surrounding expression or declaration: `"%select{class|protocol|category|class extension|implementation"`.
  **L533 CN**: 继续构造周围的表达式或声明：`"%select{class|protocol|category|class extension|implementation"`。
- **L534 EN**: Adds a standalone statement or declaration: `"|category implementation}0 started here">;`.
  **L534 CN**: 添加一条独立语句或声明：`"|category implementation}0 started here">;`。
- **L535 EN**: Declares TableGen def record `warn_objc_protocol_qualifier_missing_id`.
  **L535 CN**: 声明 TableGen def 记录 `warn_objc_protocol_qualifier_missing_id`。
- **L536 EN**: Adds a standalone statement or declaration: `"protocol has no object type specified; defaults to qualified 'id'">;`.
  **L536 CN**: 添加一条独立语句或声明：`"protocol has no object type specified; defaults to qualified 'id'">;`。
- **L537 EN**: Declares TableGen def record `err_objc_unknown_at`.
  **L537 CN**: 声明 TableGen def 记录 `err_objc_unknown_at`。
- **L538 EN**: Declares TableGen def record `err_illegal_super_cast`.
  **L538 CN**: 声明 TableGen def 记录 `err_illegal_super_cast`。
- **L539 EN**: Executes a call or declaration centered on `'super'`.
  **L539 CN**: 执行以 `'super'` 为核心的调用或声明。
- **L540 EN**: Declares TableGen def record `err_nsnumber_nonliteral_unary`.
  **L540 CN**: 声明 TableGen def 记录 `err_nsnumber_nonliteral_unary`。
- **L541 EN**: Adds a standalone statement or declaration: `"@%0 must be followed by a number to form an NSNumber object">;`.
  **L541 CN**: 添加一条独立语句或声明：`"@%0 must be followed by a number to form an NSNumber object">;`。
- **L542 EN**: Declares TableGen def record `warn_cstyle_param`.
  **L542 CN**: 声明 TableGen def 记录 `warn_cstyle_param`。
- **L543 EN**: Continues the surrounding expression or declaration: `"use of C-style parameters in Objective-C method declarations"`.
  **L543 CN**: 继续构造周围的表达式或声明：`"use of C-style parameters in Objective-C method declarations"`。
- **L544 EN**: Adds a standalone statement or declaration: `" is deprecated">, InGroup<DeprecatedDeclarations>;`.
  **L544 CN**: 添加一条独立语句或声明：`" is deprecated">, InGroup<DeprecatedDeclarations>;`。
- **L545 EN**: Blank line separating nearby declarations or logic blocks.
  **L545 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L546 EN**: Assigns a TableGen property that affects following records or inherited fields: `let CategoryName = "ARC Parse Issue" in {`.
  **L546 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let CategoryName = "ARC Parse Issue" in {`。
- **L547 EN**: Declares TableGen def record `err_arc_bridge_retain`.
  **L547 CN**: 声明 TableGen def 记录 `err_arc_bridge_retain`。
- **L548 EN**: Adds a standalone statement or declaration: `"unknown cast annotation __bridge_retain; did you mean __bridge_retained?">;`.
  **L548 CN**: 添加一条独立语句或声明：`"unknown cast annotation __bridge_retain; did you mean __bridge_retained?">;`。
- **L549 EN**: Comment explains nearby logic, constraints, or intent: `To be default mapped to an error later.`.
  **L549 CN**: 注释解释附近代码的逻辑、约束或设计意图：`To be default mapped to an error later.`。
- **L550 EN**: Declares TableGen def record `warn_arc_bridge_cast_nonarc`.
  **L550 CN**: 声明 TableGen def 记录 `warn_arc_bridge_cast_nonarc`。
- **L551 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'%0' casts have no effect when not using ARC">,`.
  **L551 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'%0' casts have no effect when not using ARC">,`。
- **L552 EN**: Adds a standalone statement or declaration: `InGroup<DiagGroup<"arc-bridge-casts-disallowed-in-nonarc">>;`.
  **L552 CN**: 添加一条独立语句或声明：`InGroup<DiagGroup<"arc-bridge-casts-disallowed-in-nonarc">>;`。

### Lines 553-576

````tablegen
}

def err_objc_illegal_visibility_spec : Error<
  "illegal visibility specification">;
def err_objc_illegal_interface_qual : Error<"illegal interface qualifier">;
def err_objc_expected_equal_for_getter : Error<
  "expected '=' for Objective-C getter">;
def err_objc_expected_equal_for_setter : Error<
  "expected '=' for Objective-C setter">;
def err_objc_expected_selector_for_getter_setter : Error<
  "expected selector for Objective-C %select{setter|getter}0">;
def err_objc_property_requires_field_name : Error<
  "property requires fields to be named">;
def err_objc_property_bitfield : Error<"property name cannot be a bit-field">;
def err_objc_expected_property_attr : Error<"unknown property attribute %0">;
def err_objc_unexpected_attr : Error<
  "prefix attribute must be followed by an interface, protocol, or implementation">;
def err_objc_postfix_attribute : Error <
  "postfix attributes are not allowed on Objective-C directives">;
def err_objc_postfix_attribute_hint : Error <
  "postfix attributes are not allowed on Objective-C directives, place"
  " them in front of '%select{@interface|@protocol}0'">;
def err_objc_directive_only_in_protocol : Error<
  "directive may only be specified in protocols only">;
````
- **L553 EN**: Closes the current lexical scope or compound statement.
  **L553 CN**: 结束当前词法作用域或复合语句块。
- **L554 EN**: Blank line separating nearby declarations or logic blocks.
  **L554 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L555 EN**: Declares TableGen def record `err_objc_illegal_visibility_spec`.
  **L555 CN**: 声明 TableGen def 记录 `err_objc_illegal_visibility_spec`。
- **L556 EN**: Adds a standalone statement or declaration: `"illegal visibility specification">;`.
  **L556 CN**: 添加一条独立语句或声明：`"illegal visibility specification">;`。
- **L557 EN**: Declares TableGen def record `err_objc_illegal_interface_qual`.
  **L557 CN**: 声明 TableGen def 记录 `err_objc_illegal_interface_qual`。
- **L558 EN**: Declares TableGen def record `err_objc_expected_equal_for_getter`.
  **L558 CN**: 声明 TableGen def 记录 `err_objc_expected_equal_for_getter`。
- **L559 EN**: Adds a standalone statement or declaration: `"expected '=' for Objective-C getter">;`.
  **L559 CN**: 添加一条独立语句或声明：`"expected '=' for Objective-C getter">;`。
- **L560 EN**: Declares TableGen def record `err_objc_expected_equal_for_setter`.
  **L560 CN**: 声明 TableGen def 记录 `err_objc_expected_equal_for_setter`。
- **L561 EN**: Adds a standalone statement or declaration: `"expected '=' for Objective-C setter">;`.
  **L561 CN**: 添加一条独立语句或声明：`"expected '=' for Objective-C setter">;`。
- **L562 EN**: Declares TableGen def record `err_objc_expected_selector_for_getter_setter`.
  **L562 CN**: 声明 TableGen def 记录 `err_objc_expected_selector_for_getter_setter`。
- **L563 EN**: Adds a standalone statement or declaration: `"expected selector for Objective-C %select{setter|getter}0">;`.
  **L563 CN**: 添加一条独立语句或声明：`"expected selector for Objective-C %select{setter|getter}0">;`。
- **L564 EN**: Declares TableGen def record `err_objc_property_requires_field_name`.
  **L564 CN**: 声明 TableGen def 记录 `err_objc_property_requires_field_name`。
- **L565 EN**: Adds a standalone statement or declaration: `"property requires fields to be named">;`.
  **L565 CN**: 添加一条独立语句或声明：`"property requires fields to be named">;`。
- **L566 EN**: Declares TableGen def record `err_objc_property_bitfield`.
  **L566 CN**: 声明 TableGen def 记录 `err_objc_property_bitfield`。
- **L567 EN**: Declares TableGen def record `err_objc_expected_property_attr`.
  **L567 CN**: 声明 TableGen def 记录 `err_objc_expected_property_attr`。
- **L568 EN**: Declares TableGen def record `err_objc_unexpected_attr`.
  **L568 CN**: 声明 TableGen def 记录 `err_objc_unexpected_attr`。
- **L569 EN**: Adds a standalone statement or declaration: `"prefix attribute must be followed by an interface, protocol, or implementation">;`.
  **L569 CN**: 添加一条独立语句或声明：`"prefix attribute must be followed by an interface, protocol, or implementation">;`。
- **L570 EN**: Declares TableGen def record `err_objc_postfix_attribute`.
  **L570 CN**: 声明 TableGen def 记录 `err_objc_postfix_attribute`。
- **L571 EN**: Adds a standalone statement or declaration: `"postfix attributes are not allowed on Objective-C directives">;`.
  **L571 CN**: 添加一条独立语句或声明：`"postfix attributes are not allowed on Objective-C directives">;`。
- **L572 EN**: Declares TableGen def record `err_objc_postfix_attribute_hint`.
  **L572 CN**: 声明 TableGen def 记录 `err_objc_postfix_attribute_hint`。
- **L573 EN**: Continues the surrounding expression or declaration: `"postfix attributes are not allowed on Objective-C directives, place"`.
  **L573 CN**: 继续构造周围的表达式或声明：`"postfix attributes are not allowed on Objective-C directives, place"`。
- **L574 EN**: Adds a standalone statement or declaration: `" them in front of '%select{@interface|@protocol}0'">;`.
  **L574 CN**: 添加一条独立语句或声明：`" them in front of '%select{@interface|@protocol}0'">;`。
- **L575 EN**: Declares TableGen def record `err_objc_directive_only_in_protocol`.
  **L575 CN**: 声明 TableGen def 记录 `err_objc_directive_only_in_protocol`。
- **L576 EN**: Adds a standalone statement or declaration: `"directive may only be specified in protocols only">;`.
  **L576 CN**: 添加一条独立语句或声明：`"directive may only be specified in protocols only">;`。

### Lines 577-600

````tablegen
def err_missing_catch_finally : Error<
  "@try statement without a @catch and @finally clause">;
def err_objc_concat_string : Error<"unexpected token after Objective-C string">;
def err_expected_objc_container : Error<
  "'@end' must appear in an Objective-C context">;
def err_unexpected_protocol_qualifier : Error<
  "@implementation declaration cannot be protocol qualified">;
def err_objc_unexpected_atend : Error<
  "'@end' appears where closing brace '}' is expected">;
def err_synthesized_property_name : Error<
  "expected a property name in @synthesize">;
def warn_semicolon_before_method_body : Warning<
  "semicolon before method body is ignored">,
  InGroup<SemiBeforeMethodBody>, DefaultIgnore;
def note_extra_comma_message_arg : Note<
  "comma separating Objective-C messaging arguments">;

def err_expected_field_designator : Error<
  "expected a field designator, such as '.field = 4'">;

def err_declaration_does_not_declare_param : Error<
  "declaration does not declare a parameter">;
def err_no_matching_param : Error<"parameter named %0 is missing">;

````
- **L577 EN**: Declares TableGen def record `err_missing_catch_finally`.
  **L577 CN**: 声明 TableGen def 记录 `err_missing_catch_finally`。
- **L578 EN**: Adds a standalone statement or declaration: `"@try statement without a @catch and @finally clause">;`.
  **L578 CN**: 添加一条独立语句或声明：`"@try statement without a @catch and @finally clause">;`。
- **L579 EN**: Declares TableGen def record `err_objc_concat_string`.
  **L579 CN**: 声明 TableGen def 记录 `err_objc_concat_string`。
- **L580 EN**: Declares TableGen def record `err_expected_objc_container`.
  **L580 CN**: 声明 TableGen def 记录 `err_expected_objc_container`。
- **L581 EN**: Adds a standalone statement or declaration: `"'@end' must appear in an Objective-C context">;`.
  **L581 CN**: 添加一条独立语句或声明：`"'@end' must appear in an Objective-C context">;`。
- **L582 EN**: Declares TableGen def record `err_unexpected_protocol_qualifier`.
  **L582 CN**: 声明 TableGen def 记录 `err_unexpected_protocol_qualifier`。
- **L583 EN**: Adds a standalone statement or declaration: `"@implementation declaration cannot be protocol qualified">;`.
  **L583 CN**: 添加一条独立语句或声明：`"@implementation declaration cannot be protocol qualified">;`。
- **L584 EN**: Declares TableGen def record `err_objc_unexpected_atend`.
  **L584 CN**: 声明 TableGen def 记录 `err_objc_unexpected_atend`。
- **L585 EN**: Adds a standalone statement or declaration: `"'@end' appears where closing brace '}' is expected">;`.
  **L585 CN**: 添加一条独立语句或声明：`"'@end' appears where closing brace '}' is expected">;`。
- **L586 EN**: Declares TableGen def record `err_synthesized_property_name`.
  **L586 CN**: 声明 TableGen def 记录 `err_synthesized_property_name`。
- **L587 EN**: Adds a standalone statement or declaration: `"expected a property name in @synthesize">;`.
  **L587 CN**: 添加一条独立语句或声明：`"expected a property name in @synthesize">;`。
- **L588 EN**: Declares TableGen def record `warn_semicolon_before_method_body`.
  **L588 CN**: 声明 TableGen def 记录 `warn_semicolon_before_method_body`。
- **L589 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"semicolon before method body is ignored">,`.
  **L589 CN**: 继续一个多行参数列表、初始化器或聚合项：`"semicolon before method body is ignored">,`。
- **L590 EN**: Adds a standalone statement or declaration: `InGroup<SemiBeforeMethodBody>, DefaultIgnore;`.
  **L590 CN**: 添加一条独立语句或声明：`InGroup<SemiBeforeMethodBody>, DefaultIgnore;`。
- **L591 EN**: Declares TableGen def record `note_extra_comma_message_arg`.
  **L591 CN**: 声明 TableGen def 记录 `note_extra_comma_message_arg`。
- **L592 EN**: Adds a standalone statement or declaration: `"comma separating Objective-C messaging arguments">;`.
  **L592 CN**: 添加一条独立语句或声明：`"comma separating Objective-C messaging arguments">;`。
- **L593 EN**: Blank line separating nearby declarations or logic blocks.
  **L593 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L594 EN**: Declares TableGen def record `err_expected_field_designator`.
  **L594 CN**: 声明 TableGen def 记录 `err_expected_field_designator`。
- **L595 EN**: Adds a standalone statement or declaration: `"expected a field designator, such as '.field = 4'">;`.
  **L595 CN**: 添加一条独立语句或声明：`"expected a field designator, such as '.field = 4'">;`。
- **L596 EN**: Blank line separating nearby declarations or logic blocks.
  **L596 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L597 EN**: Declares TableGen def record `err_declaration_does_not_declare_param`.
  **L597 CN**: 声明 TableGen def 记录 `err_declaration_does_not_declare_param`。
- **L598 EN**: Adds a standalone statement or declaration: `"declaration does not declare a parameter">;`.
  **L598 CN**: 添加一条独立语句或声明：`"declaration does not declare a parameter">;`。
- **L599 EN**: Declares TableGen def record `err_no_matching_param`.
  **L599 CN**: 声明 TableGen def 记录 `err_no_matching_param`。
- **L600 EN**: Blank line separating nearby declarations or logic blocks.
  **L600 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 601-624

````tablegen
/// Objective-C++ parser diagnostics
def err_expected_token_instead_of_objcxx_keyword : Error<
  "expected %0; %1 is a keyword in Objective-C++">;
def err_expected_member_name_or_semi_objcxx_keyword : Error<
  "expected member name or ';' after declaration specifiers; "
  "%0 is a keyword in Objective-C++">;

/// C++ parser diagnostics
def err_invalid_operator_on_type : Error<
  "cannot use %select{dot|arrow}0 operator on a type">;
def err_expected_unqualified_id : Error<
  "expected %select{identifier|unqualified-id}0">;
def err_while_loop_outside_of_a_function : Error<
  "while loop outside of a function">;
def err_brackets_go_after_unqualified_id : Error<
  "brackets are not allowed here; to declare an array, "
  "place the brackets after the %select{identifier|name}0">;
def err_unexpected_unqualified_id : Error<"type-id cannot have a name">;
def err_func_def_no_params : Error<
  "function definition does not declare parameters">;
def err_expected_lparen_after_type : Error<
  "expected '(' for function-style cast or type construction">;
def err_expected_init_in_condition : Error<
  "variable declaration in condition must have an initializer">;
````
- **L601 EN**: Comment explains nearby logic, constraints, or intent: `Objective-C++ parser diagnostics`.
  **L601 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Objective-C++ parser diagnostics`。
- **L602 EN**: Declares TableGen def record `err_expected_token_instead_of_objcxx_keyword`.
  **L602 CN**: 声明 TableGen def 记录 `err_expected_token_instead_of_objcxx_keyword`。
- **L603 EN**: Adds a standalone statement or declaration: `"expected %0; %1 is a keyword in Objective-C++">;`.
  **L603 CN**: 添加一条独立语句或声明：`"expected %0; %1 is a keyword in Objective-C++">;`。
- **L604 EN**: Declares TableGen def record `err_expected_member_name_or_semi_objcxx_keyword`.
  **L604 CN**: 声明 TableGen def 记录 `err_expected_member_name_or_semi_objcxx_keyword`。
- **L605 EN**: Continues the surrounding expression or declaration: `"expected member name or ';' after declaration specifiers; "`.
  **L605 CN**: 继续构造周围的表达式或声明：`"expected member name or ';' after declaration specifiers; "`。
- **L606 EN**: Adds a standalone statement or declaration: `"%0 is a keyword in Objective-C++">;`.
  **L606 CN**: 添加一条独立语句或声明：`"%0 is a keyword in Objective-C++">;`。
- **L607 EN**: Blank line separating nearby declarations or logic blocks.
  **L607 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L608 EN**: Comment explains nearby logic, constraints, or intent: `C++ parser diagnostics`.
  **L608 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C++ parser diagnostics`。
- **L609 EN**: Declares TableGen def record `err_invalid_operator_on_type`.
  **L609 CN**: 声明 TableGen def 记录 `err_invalid_operator_on_type`。
- **L610 EN**: Adds a standalone statement or declaration: `"cannot use %select{dot|arrow}0 operator on a type">;`.
  **L610 CN**: 添加一条独立语句或声明：`"cannot use %select{dot|arrow}0 operator on a type">;`。
- **L611 EN**: Declares TableGen def record `err_expected_unqualified_id`.
  **L611 CN**: 声明 TableGen def 记录 `err_expected_unqualified_id`。
- **L612 EN**: Adds a standalone statement or declaration: `"expected %select{identifier|unqualified-id}0">;`.
  **L612 CN**: 添加一条独立语句或声明：`"expected %select{identifier|unqualified-id}0">;`。
- **L613 EN**: Declares TableGen def record `err_while_loop_outside_of_a_function`.
  **L613 CN**: 声明 TableGen def 记录 `err_while_loop_outside_of_a_function`。
- **L614 EN**: Adds a standalone statement or declaration: `"while loop outside of a function">;`.
  **L614 CN**: 添加一条独立语句或声明：`"while loop outside of a function">;`。
- **L615 EN**: Declares TableGen def record `err_brackets_go_after_unqualified_id`.
  **L615 CN**: 声明 TableGen def 记录 `err_brackets_go_after_unqualified_id`。
- **L616 EN**: Continues the surrounding expression or declaration: `"brackets are not allowed here; to declare an array, "`.
  **L616 CN**: 继续构造周围的表达式或声明：`"brackets are not allowed here; to declare an array, "`。
- **L617 EN**: Adds a standalone statement or declaration: `"place the brackets after the %select{identifier|name}0">;`.
  **L617 CN**: 添加一条独立语句或声明：`"place the brackets after the %select{identifier|name}0">;`。
- **L618 EN**: Declares TableGen def record `err_unexpected_unqualified_id`.
  **L618 CN**: 声明 TableGen def 记录 `err_unexpected_unqualified_id`。
- **L619 EN**: Declares TableGen def record `err_func_def_no_params`.
  **L619 CN**: 声明 TableGen def 记录 `err_func_def_no_params`。
- **L620 EN**: Adds a standalone statement or declaration: `"function definition does not declare parameters">;`.
  **L620 CN**: 添加一条独立语句或声明：`"function definition does not declare parameters">;`。
- **L621 EN**: Declares TableGen def record `err_expected_lparen_after_type`.
  **L621 CN**: 声明 TableGen def 记录 `err_expected_lparen_after_type`。
- **L622 EN**: Executes a call or declaration centered on `'`.
  **L622 CN**: 执行以 `'` 为核心的调用或声明。
- **L623 EN**: Declares TableGen def record `err_expected_init_in_condition`.
  **L623 CN**: 声明 TableGen def 记录 `err_expected_init_in_condition`。
- **L624 EN**: Adds a standalone statement or declaration: `"variable declaration in condition must have an initializer">;`.
  **L624 CN**: 添加一条独立语句或声明：`"variable declaration in condition must have an initializer">;`。

### Lines 625-648

````tablegen
def err_expected_init_in_condition_lparen : Error<
  "variable declaration in condition cannot have a parenthesized initializer">;
def err_extraneous_rparen_in_condition : Error<
  "extraneous ')' after condition, expected a statement">;
def ext_alias_in_init_statement : ExtWarn<
  "alias declaration in this context is a C++23 extension">,
  InGroup<CXX23>;
def warn_cxx20_alias_in_init_statement  : Warning<
  "alias declaration in this context is incompatible with C++ standards before C++23">,
  DefaultIgnore, InGroup<CXXPre23Compat>;
def warn_dangling_else : Warning<
  "add explicit braces to avoid dangling else">,
  InGroup<DanglingElse>;
def err_expected_member_or_base_name : Error<
  "expected class member or base class name">;
def err_expected_lbrace_after_base_specifiers : Error<
  "expected '{' after base class list">;
def err_missing_end_of_definition : Error<
  "missing '}' at end of definition of %q0">;
def note_missing_end_of_definition_before : Note<
  "still within definition of %q0 here">;
def ext_ellipsis_exception_spec : Extension<
  "exception specification of '...' is a Microsoft extension">,
  InGroup<MicrosoftExceptionSpec>;
````
- **L625 EN**: Declares TableGen def record `err_expected_init_in_condition_lparen`.
  **L625 CN**: 声明 TableGen def 记录 `err_expected_init_in_condition_lparen`。
- **L626 EN**: Adds a standalone statement or declaration: `"variable declaration in condition cannot have a parenthesized initializer">;`.
  **L626 CN**: 添加一条独立语句或声明：`"variable declaration in condition cannot have a parenthesized initializer">;`。
- **L627 EN**: Declares TableGen def record `err_extraneous_rparen_in_condition`.
  **L627 CN**: 声明 TableGen def 记录 `err_extraneous_rparen_in_condition`。
- **L628 EN**: Adds a standalone statement or declaration: `"extraneous ')' after condition, expected a statement">;`.
  **L628 CN**: 添加一条独立语句或声明：`"extraneous ')' after condition, expected a statement">;`。
- **L629 EN**: Declares TableGen def record `ext_alias_in_init_statement`.
  **L629 CN**: 声明 TableGen def 记录 `ext_alias_in_init_statement`。
- **L630 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"alias declaration in this context is a C++23 extension">,`.
  **L630 CN**: 继续一个多行参数列表、初始化器或聚合项：`"alias declaration in this context is a C++23 extension">,`。
- **L631 EN**: Adds a standalone statement or declaration: `InGroup<CXX23>;`.
  **L631 CN**: 添加一条独立语句或声明：`InGroup<CXX23>;`。
- **L632 EN**: Declares TableGen def record `warn_cxx20_alias_in_init_statement`.
  **L632 CN**: 声明 TableGen def 记录 `warn_cxx20_alias_in_init_statement`。
- **L633 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"alias declaration in this context is incompatible with C++ standards before C++23">,`.
  **L633 CN**: 继续一个多行参数列表、初始化器或聚合项：`"alias declaration in this context is incompatible with C++ standards before C++23">,`。
- **L634 EN**: Adds a standalone statement or declaration: `DefaultIgnore, InGroup<CXXPre23Compat>;`.
  **L634 CN**: 添加一条独立语句或声明：`DefaultIgnore, InGroup<CXXPre23Compat>;`。
- **L635 EN**: Declares TableGen def record `warn_dangling_else`.
  **L635 CN**: 声明 TableGen def 记录 `warn_dangling_else`。
- **L636 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"add explicit braces to avoid dangling else">,`.
  **L636 CN**: 继续一个多行参数列表、初始化器或聚合项：`"add explicit braces to avoid dangling else">,`。
- **L637 EN**: Adds a standalone statement or declaration: `InGroup<DanglingElse>;`.
  **L637 CN**: 添加一条独立语句或声明：`InGroup<DanglingElse>;`。
- **L638 EN**: Declares TableGen def record `err_expected_member_or_base_name`.
  **L638 CN**: 声明 TableGen def 记录 `err_expected_member_or_base_name`。
- **L639 EN**: Adds a standalone statement or declaration: `"expected class member or base class name">;`.
  **L639 CN**: 添加一条独立语句或声明：`"expected class member or base class name">;`。
- **L640 EN**: Declares TableGen def record `err_expected_lbrace_after_base_specifiers`.
  **L640 CN**: 声明 TableGen def 记录 `err_expected_lbrace_after_base_specifiers`。
- **L641 EN**: Adds a standalone statement or declaration: `"expected '{' after base class list">;`.
  **L641 CN**: 添加一条独立语句或声明：`"expected '{' after base class list">;`。
- **L642 EN**: Declares TableGen def record `err_missing_end_of_definition`.
  **L642 CN**: 声明 TableGen def 记录 `err_missing_end_of_definition`。
- **L643 EN**: Adds a standalone statement or declaration: `"missing '}' at end of definition of %q0">;`.
  **L643 CN**: 添加一条独立语句或声明：`"missing '}' at end of definition of %q0">;`。
- **L644 EN**: Declares TableGen def record `note_missing_end_of_definition_before`.
  **L644 CN**: 声明 TableGen def 记录 `note_missing_end_of_definition_before`。
- **L645 EN**: Adds a standalone statement or declaration: `"still within definition of %q0 here">;`.
  **L645 CN**: 添加一条独立语句或声明：`"still within definition of %q0 here">;`。
- **L646 EN**: Declares TableGen def record `ext_ellipsis_exception_spec`.
  **L646 CN**: 声明 TableGen def 记录 `ext_ellipsis_exception_spec`。
- **L647 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"exception specification of '...' is a Microsoft extension">,`.
  **L647 CN**: 继续一个多行参数列表、初始化器或聚合项：`"exception specification of '...' is a Microsoft extension">,`。
- **L648 EN**: Adds a standalone statement or declaration: `InGroup<MicrosoftExceptionSpec>;`.
  **L648 CN**: 添加一条独立语句或声明：`InGroup<MicrosoftExceptionSpec>;`。

### Lines 649-672

````tablegen
def err_dynamic_and_noexcept_specification : Error<
  "cannot have both throw() and noexcept() clause on the same function">;
def err_except_spec_unparsed : Error<
  "unexpected end of exception specification">;
def ext_dynamic_exception_spec : ExtWarn<
  "ISO C++17 does not allow dynamic exception specifications">,
  InGroup<DynamicExceptionSpec>, DefaultError;
def warn_exception_spec_deprecated : Warning<
  "dynamic exception specifications are deprecated">,
  InGroup<DeprecatedDynamicExceptionSpec>, DefaultIgnore;
def note_exception_spec_deprecated : Note<"use '%0' instead">;
def warn_cxx98_compat_noexcept_decl : Warning<
  "noexcept specifications are incompatible with C++98">,
  InGroup<CXX98Compat>, DefaultIgnore;
def err_expected_catch : Error<"expected catch">;
def err_using_namespace_in_class : Error<
  "'using namespace' is not allowed in classes">;
def warn_cxx17_compat_using_enum_declaration : Warning<
  "using enum declaration is incompatible with C++ standards before C++20">,
  InGroup<CXXPre20Compat>, DefaultIgnore;
def ext_using_enum_declaration : ExtWarn<
  "using enum declaration is a C++20 extension">,
  InGroup<CXX20>;
def err_using_enum_expect_identifier : Error<
````
- **L649 EN**: Declares TableGen def record `err_dynamic_and_noexcept_specification`.
  **L649 CN**: 声明 TableGen def 记录 `err_dynamic_and_noexcept_specification`。
- **L650 EN**: Executes a call or declaration centered on `throw`.
  **L650 CN**: 执行以 `throw` 为核心的调用或声明。
- **L651 EN**: Declares TableGen def record `err_except_spec_unparsed`.
  **L651 CN**: 声明 TableGen def 记录 `err_except_spec_unparsed`。
- **L652 EN**: Adds a standalone statement or declaration: `"unexpected end of exception specification">;`.
  **L652 CN**: 添加一条独立语句或声明：`"unexpected end of exception specification">;`。
- **L653 EN**: Declares TableGen def record `ext_dynamic_exception_spec`.
  **L653 CN**: 声明 TableGen def 记录 `ext_dynamic_exception_spec`。
- **L654 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"ISO C++17 does not allow dynamic exception specifications">,`.
  **L654 CN**: 继续一个多行参数列表、初始化器或聚合项：`"ISO C++17 does not allow dynamic exception specifications">,`。
- **L655 EN**: Adds a standalone statement or declaration: `InGroup<DynamicExceptionSpec>, DefaultError;`.
  **L655 CN**: 添加一条独立语句或声明：`InGroup<DynamicExceptionSpec>, DefaultError;`。
- **L656 EN**: Declares TableGen def record `warn_exception_spec_deprecated`.
  **L656 CN**: 声明 TableGen def 记录 `warn_exception_spec_deprecated`。
- **L657 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"dynamic exception specifications are deprecated">,`.
  **L657 CN**: 继续一个多行参数列表、初始化器或聚合项：`"dynamic exception specifications are deprecated">,`。
- **L658 EN**: Adds a standalone statement or declaration: `InGroup<DeprecatedDynamicExceptionSpec>, DefaultIgnore;`.
  **L658 CN**: 添加一条独立语句或声明：`InGroup<DeprecatedDynamicExceptionSpec>, DefaultIgnore;`。
- **L659 EN**: Declares TableGen def record `note_exception_spec_deprecated`.
  **L659 CN**: 声明 TableGen def 记录 `note_exception_spec_deprecated`。
- **L660 EN**: Declares TableGen def record `warn_cxx98_compat_noexcept_decl`.
  **L660 CN**: 声明 TableGen def 记录 `warn_cxx98_compat_noexcept_decl`。
- **L661 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"noexcept specifications are incompatible with C++98">,`.
  **L661 CN**: 继续一个多行参数列表、初始化器或聚合项：`"noexcept specifications are incompatible with C++98">,`。
- **L662 EN**: Adds a standalone statement or declaration: `InGroup<CXX98Compat>, DefaultIgnore;`.
  **L662 CN**: 添加一条独立语句或声明：`InGroup<CXX98Compat>, DefaultIgnore;`。
- **L663 EN**: Declares TableGen def record `err_expected_catch`.
  **L663 CN**: 声明 TableGen def 记录 `err_expected_catch`。
- **L664 EN**: Declares TableGen def record `err_using_namespace_in_class`.
  **L664 CN**: 声明 TableGen def 记录 `err_using_namespace_in_class`。
- **L665 EN**: Adds a standalone statement or declaration: `"'using namespace' is not allowed in classes">;`.
  **L665 CN**: 添加一条独立语句或声明：`"'using namespace' is not allowed in classes">;`。
- **L666 EN**: Declares TableGen def record `warn_cxx17_compat_using_enum_declaration`.
  **L666 CN**: 声明 TableGen def 记录 `warn_cxx17_compat_using_enum_declaration`。
- **L667 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"using enum declaration is incompatible with C++ standards before C++20">,`.
  **L667 CN**: 继续一个多行参数列表、初始化器或聚合项：`"using enum declaration is incompatible with C++ standards before C++20">,`。
- **L668 EN**: Adds a standalone statement or declaration: `InGroup<CXXPre20Compat>, DefaultIgnore;`.
  **L668 CN**: 添加一条独立语句或声明：`InGroup<CXXPre20Compat>, DefaultIgnore;`。
- **L669 EN**: Declares TableGen def record `ext_using_enum_declaration`.
  **L669 CN**: 声明 TableGen def 记录 `ext_using_enum_declaration`。
- **L670 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"using enum declaration is a C++20 extension">,`.
  **L670 CN**: 继续一个多行参数列表、初始化器或聚合项：`"using enum declaration is a C++20 extension">,`。
- **L671 EN**: Adds a standalone statement or declaration: `InGroup<CXX20>;`.
  **L671 CN**: 添加一条独立语句或声明：`InGroup<CXX20>;`。
- **L672 EN**: Declares TableGen def record `err_using_enum_expect_identifier`.
  **L672 CN**: 声明 TableGen def 记录 `err_using_enum_expect_identifier`。

### Lines 673-696

````tablegen
  "using enum %select{requires an enum or typedef name|"
  "does not permit an elaborated enum specifier}0">;
def err_constructor_bad_name : Error<
  "missing return type for function %0; did you mean the constructor name %1?">;
def err_destructor_tilde_identifier : Error<
  "expected a class name after '~' to name a destructor">;
def err_destructor_tilde_scope : Error<
  "'~' in destructor name should be after nested name specifier">;
def err_destructor_template_id : Error<
  "destructor name %0 does not refer to a template">;
def err_default_arg_unparsed : Error<
  "unexpected end of default argument expression">;
def err_bracket_depth_exceeded : Error<
  "bracket nesting level exceeded maximum of %0">, DefaultFatal;
def note_bracket_depth : Note<
  "use -fbracket-depth=N to increase maximum nesting level">;
def err_misplaced_ellipsis_in_declaration : Error<
  "'...' must %select{immediately precede declared identifier|"
  "be innermost component of anonymous pack declaration}0">;
def warn_misplaced_ellipsis_vararg : Warning<
  "'...' in this location creates a C-style varargs function"
  "%select{, not a function parameter pack|}0">,
  InGroup<DiagGroup<"ambiguous-ellipsis">>;
def note_misplaced_ellipsis_vararg_existing_ellipsis : Note<
````
- **L673 EN**: Continues the surrounding expression or declaration: `"using enum %select{requires an enum or typedef name|"`.
  **L673 CN**: 继续构造周围的表达式或声明：`"using enum %select{requires an enum or typedef name|"`。
- **L674 EN**: Adds a standalone statement or declaration: `"does not permit an elaborated enum specifier}0">;`.
  **L674 CN**: 添加一条独立语句或声明：`"does not permit an elaborated enum specifier}0">;`。
- **L675 EN**: Declares TableGen def record `err_constructor_bad_name`.
  **L675 CN**: 声明 TableGen def 记录 `err_constructor_bad_name`。
- **L676 EN**: Adds a standalone statement or declaration: `"missing return type for function %0; did you mean the constructor name %1?">;`.
  **L676 CN**: 添加一条独立语句或声明：`"missing return type for function %0; did you mean the constructor name %1?">;`。
- **L677 EN**: Declares TableGen def record `err_destructor_tilde_identifier`.
  **L677 CN**: 声明 TableGen def 记录 `err_destructor_tilde_identifier`。
- **L678 EN**: Adds a standalone statement or declaration: `"expected a class name after '~' to name a destructor">;`.
  **L678 CN**: 添加一条独立语句或声明：`"expected a class name after '~' to name a destructor">;`。
- **L679 EN**: Declares TableGen def record `err_destructor_tilde_scope`.
  **L679 CN**: 声明 TableGen def 记录 `err_destructor_tilde_scope`。
- **L680 EN**: Adds a standalone statement or declaration: `"'~' in destructor name should be after nested name specifier">;`.
  **L680 CN**: 添加一条独立语句或声明：`"'~' in destructor name should be after nested name specifier">;`。
- **L681 EN**: Declares TableGen def record `err_destructor_template_id`.
  **L681 CN**: 声明 TableGen def 记录 `err_destructor_template_id`。
- **L682 EN**: Adds a standalone statement or declaration: `"destructor name %0 does not refer to a template">;`.
  **L682 CN**: 添加一条独立语句或声明：`"destructor name %0 does not refer to a template">;`。
- **L683 EN**: Declares TableGen def record `err_default_arg_unparsed`.
  **L683 CN**: 声明 TableGen def 记录 `err_default_arg_unparsed`。
- **L684 EN**: Adds a standalone statement or declaration: `"unexpected end of default argument expression">;`.
  **L684 CN**: 添加一条独立语句或声明：`"unexpected end of default argument expression">;`。
- **L685 EN**: Declares TableGen def record `err_bracket_depth_exceeded`.
  **L685 CN**: 声明 TableGen def 记录 `err_bracket_depth_exceeded`。
- **L686 EN**: Adds a standalone statement or declaration: `"bracket nesting level exceeded maximum of %0">, DefaultFatal;`.
  **L686 CN**: 添加一条独立语句或声明：`"bracket nesting level exceeded maximum of %0">, DefaultFatal;`。
- **L687 EN**: Declares TableGen def record `note_bracket_depth`.
  **L687 CN**: 声明 TableGen def 记录 `note_bracket_depth`。
- **L688 EN**: Adds a standalone statement or declaration: `"use -fbracket-depth=N to increase maximum nesting level">;`.
  **L688 CN**: 添加一条独立语句或声明：`"use -fbracket-depth=N to increase maximum nesting level">;`。
- **L689 EN**: Declares TableGen def record `err_misplaced_ellipsis_in_declaration`.
  **L689 CN**: 声明 TableGen def 记录 `err_misplaced_ellipsis_in_declaration`。
- **L690 EN**: Continues the surrounding expression or declaration: `"'...' must %select{immediately precede declared identifier|"`.
  **L690 CN**: 继续构造周围的表达式或声明：`"'...' must %select{immediately precede declared identifier|"`。
- **L691 EN**: Adds a standalone statement or declaration: `"be innermost component of anonymous pack declaration}0">;`.
  **L691 CN**: 添加一条独立语句或声明：`"be innermost component of anonymous pack declaration}0">;`。
- **L692 EN**: Declares TableGen def record `warn_misplaced_ellipsis_vararg`.
  **L692 CN**: 声明 TableGen def 记录 `warn_misplaced_ellipsis_vararg`。
- **L693 EN**: Continues the surrounding expression or declaration: `"'...' in this location creates a C-style varargs function"`.
  **L693 CN**: 继续构造周围的表达式或声明：`"'...' in this location creates a C-style varargs function"`。
- **L694 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"%select{, not a function parameter pack|}0">,`.
  **L694 CN**: 继续一个多行参数列表、初始化器或聚合项：`"%select{, not a function parameter pack|}0">,`。
- **L695 EN**: Adds a standalone statement or declaration: `InGroup<DiagGroup<"ambiguous-ellipsis">>;`.
  **L695 CN**: 添加一条独立语句或声明：`InGroup<DiagGroup<"ambiguous-ellipsis">>;`。
- **L696 EN**: Declares TableGen def record `note_misplaced_ellipsis_vararg_existing_ellipsis`.
  **L696 CN**: 声明 TableGen def 记录 `note_misplaced_ellipsis_vararg_existing_ellipsis`。

### Lines 697-720

````tablegen
  "preceding '...' declares a function parameter pack">;
def note_misplaced_ellipsis_vararg_add_ellipsis : Note<
  "place '...' %select{immediately before declared identifier|here}0 "
  "to declare a function parameter pack">;
def note_misplaced_ellipsis_vararg_add_comma : Note<
  "insert ',' before '...' to silence this warning">;
def ext_abstract_pack_declarator_parens : ExtWarn<
  "ISO C++11 requires a parenthesized pack declaration to have a name">,
  InGroup<DiagGroup<"anonymous-pack-parens">>;
def err_function_is_not_record : Error<
  "unexpected %0 in function call; perhaps remove the %0?">;
def err_super_in_using_declaration : Error<
  "'__super' cannot be used with a using declaration">;
def ext_constexpr_if : ExtWarn<
  "constexpr if is a C++17 extension">, InGroup<CXX17>;
def warn_cxx14_compat_constexpr_if : Warning<
  "constexpr if is incompatible with C++ standards before C++17">,
  DefaultIgnore, InGroup<CXXPre17Compat>;
def ext_consteval_if : ExtWarn<
  "consteval if is a C++23 extension">,
   InGroup<CXX23>;
def warn_cxx20_compat_consteval_if : Warning<
  "consteval if is incompatible with C++ standards before C++23">,
  InGroup<CXXPre23Compat>, DefaultIgnore;
````
- **L697 EN**: Adds a standalone statement or declaration: `"preceding '...' declares a function parameter pack">;`.
  **L697 CN**: 添加一条独立语句或声明：`"preceding '...' declares a function parameter pack">;`。
- **L698 EN**: Declares TableGen def record `note_misplaced_ellipsis_vararg_add_ellipsis`.
  **L698 CN**: 声明 TableGen def 记录 `note_misplaced_ellipsis_vararg_add_ellipsis`。
- **L699 EN**: Continues the surrounding expression or declaration: `"place '...' %select{immediately before declared identifier|here}0 "`.
  **L699 CN**: 继续构造周围的表达式或声明：`"place '...' %select{immediately before declared identifier|here}0 "`。
- **L700 EN**: Adds a standalone statement or declaration: `"to declare a function parameter pack">;`.
  **L700 CN**: 添加一条独立语句或声明：`"to declare a function parameter pack">;`。
- **L701 EN**: Declares TableGen def record `note_misplaced_ellipsis_vararg_add_comma`.
  **L701 CN**: 声明 TableGen def 记录 `note_misplaced_ellipsis_vararg_add_comma`。
- **L702 EN**: Adds a standalone statement or declaration: `"insert ',' before '...' to silence this warning">;`.
  **L702 CN**: 添加一条独立语句或声明：`"insert ',' before '...' to silence this warning">;`。
- **L703 EN**: Declares TableGen def record `ext_abstract_pack_declarator_parens`.
  **L703 CN**: 声明 TableGen def 记录 `ext_abstract_pack_declarator_parens`。
- **L704 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"ISO C++11 requires a parenthesized pack declaration to have a name">,`.
  **L704 CN**: 继续一个多行参数列表、初始化器或聚合项：`"ISO C++11 requires a parenthesized pack declaration to have a name">,`。
- **L705 EN**: Adds a standalone statement or declaration: `InGroup<DiagGroup<"anonymous-pack-parens">>;`.
  **L705 CN**: 添加一条独立语句或声明：`InGroup<DiagGroup<"anonymous-pack-parens">>;`。
- **L706 EN**: Declares TableGen def record `err_function_is_not_record`.
  **L706 CN**: 声明 TableGen def 记录 `err_function_is_not_record`。
- **L707 EN**: Adds a standalone statement or declaration: `"unexpected %0 in function call; perhaps remove the %0?">;`.
  **L707 CN**: 添加一条独立语句或声明：`"unexpected %0 in function call; perhaps remove the %0?">;`。
- **L708 EN**: Declares TableGen def record `err_super_in_using_declaration`.
  **L708 CN**: 声明 TableGen def 记录 `err_super_in_using_declaration`。
- **L709 EN**: Adds a standalone statement or declaration: `"'__super' cannot be used with a using declaration">;`.
  **L709 CN**: 添加一条独立语句或声明：`"'__super' cannot be used with a using declaration">;`。
- **L710 EN**: Declares TableGen def record `ext_constexpr_if`.
  **L710 CN**: 声明 TableGen def 记录 `ext_constexpr_if`。
- **L711 EN**: Adds a standalone statement or declaration: `"constexpr if is a C++17 extension">, InGroup<CXX17>;`.
  **L711 CN**: 添加一条独立语句或声明：`"constexpr if is a C++17 extension">, InGroup<CXX17>;`。
- **L712 EN**: Declares TableGen def record `warn_cxx14_compat_constexpr_if`.
  **L712 CN**: 声明 TableGen def 记录 `warn_cxx14_compat_constexpr_if`。
- **L713 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"constexpr if is incompatible with C++ standards before C++17">,`.
  **L713 CN**: 继续一个多行参数列表、初始化器或聚合项：`"constexpr if is incompatible with C++ standards before C++17">,`。
- **L714 EN**: Adds a standalone statement or declaration: `DefaultIgnore, InGroup<CXXPre17Compat>;`.
  **L714 CN**: 添加一条独立语句或声明：`DefaultIgnore, InGroup<CXXPre17Compat>;`。
- **L715 EN**: Declares TableGen def record `ext_consteval_if`.
  **L715 CN**: 声明 TableGen def 记录 `ext_consteval_if`。
- **L716 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"consteval if is a C++23 extension">,`.
  **L716 CN**: 继续一个多行参数列表、初始化器或聚合项：`"consteval if is a C++23 extension">,`。
- **L717 EN**: Adds a standalone statement or declaration: `InGroup<CXX23>;`.
  **L717 CN**: 添加一条独立语句或声明：`InGroup<CXX23>;`。
- **L718 EN**: Declares TableGen def record `warn_cxx20_compat_consteval_if`.
  **L718 CN**: 声明 TableGen def 记录 `warn_cxx20_compat_consteval_if`。
- **L719 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"consteval if is incompatible with C++ standards before C++23">,`.
  **L719 CN**: 继续一个多行参数列表、初始化器或聚合项：`"consteval if is incompatible with C++ standards before C++23">,`。
- **L720 EN**: Adds a standalone statement or declaration: `InGroup<CXXPre23Compat>, DefaultIgnore;`.
  **L720 CN**: 添加一条独立语句或声明：`InGroup<CXXPre23Compat>, DefaultIgnore;`。

### Lines 721-744

````tablegen

def ext_init_statement : ExtWarn<
  "'%select{if|switch}0' initialization statements are a C++17 extension">,
  InGroup<CXX17>;
def warn_cxx14_compat_init_statement : Warning<
  "%select{if|switch}0 initialization statements are incompatible with "
  "C++ standards before C++17">, DefaultIgnore, InGroup<CXXPre17Compat>;
def ext_for_range_init_stmt : ExtWarn<
  "range-based for loop initialization statements are a C++20 extension">,
  InGroup<CXX20>;
def warn_cxx17_compat_for_range_init_stmt : Warning<
  "range-based for loop initialization statements are incompatible with "
  "C++ standards before C++20">, DefaultIgnore, InGroup<CXXPre20Compat>;
def warn_empty_init_statement : Warning<
  "empty initialization statement of '%select{if|switch|range-based for}0' "
  "has no effect">, InGroup<EmptyInitStatement>, DefaultIgnore;
def err_keyword_as_parameter : Error <
  "invalid parameter name: '%0' is a keyword">;
def warn_pre_cxx26_ambiguous_pack_indexing_type : Warning<
  "%0 is no longer a pack expansion but a pack "
  "indexing type; add a name to specify a pack expansion">, InGroup<CXXPre26Compat>;

// C++ derived classes
def err_dup_virtual : Error<"duplicate 'virtual' in base specifier">;
````
- **L721 EN**: Blank line separating nearby declarations or logic blocks.
  **L721 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L722 EN**: Declares TableGen def record `ext_init_statement`.
  **L722 CN**: 声明 TableGen def 记录 `ext_init_statement`。
- **L723 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'%select{if|switch}0' initialization statements are a C++17 extension">,`.
  **L723 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'%select{if|switch}0' initialization statements are a C++17 extension">,`。
- **L724 EN**: Adds a standalone statement or declaration: `InGroup<CXX17>;`.
  **L724 CN**: 添加一条独立语句或声明：`InGroup<CXX17>;`。
- **L725 EN**: Declares TableGen def record `warn_cxx14_compat_init_statement`.
  **L725 CN**: 声明 TableGen def 记录 `warn_cxx14_compat_init_statement`。
- **L726 EN**: Continues the surrounding expression or declaration: `"%select{if|switch}0 initialization statements are incompatible with "`.
  **L726 CN**: 继续构造周围的表达式或声明：`"%select{if|switch}0 initialization statements are incompatible with "`。
- **L727 EN**: Adds a standalone statement or declaration: `"C++ standards before C++17">, DefaultIgnore, InGroup<CXXPre17Compat>;`.
  **L727 CN**: 添加一条独立语句或声明：`"C++ standards before C++17">, DefaultIgnore, InGroup<CXXPre17Compat>;`。
- **L728 EN**: Declares TableGen def record `ext_for_range_init_stmt`.
  **L728 CN**: 声明 TableGen def 记录 `ext_for_range_init_stmt`。
- **L729 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"range-based for loop initialization statements are a C++20 extension">,`.
  **L729 CN**: 继续一个多行参数列表、初始化器或聚合项：`"range-based for loop initialization statements are a C++20 extension">,`。
- **L730 EN**: Adds a standalone statement or declaration: `InGroup<CXX20>;`.
  **L730 CN**: 添加一条独立语句或声明：`InGroup<CXX20>;`。
- **L731 EN**: Declares TableGen def record `warn_cxx17_compat_for_range_init_stmt`.
  **L731 CN**: 声明 TableGen def 记录 `warn_cxx17_compat_for_range_init_stmt`。
- **L732 EN**: Continues the surrounding expression or declaration: `"range-based for loop initialization statements are incompatible with "`.
  **L732 CN**: 继续构造周围的表达式或声明：`"range-based for loop initialization statements are incompatible with "`。
- **L733 EN**: Adds a standalone statement or declaration: `"C++ standards before C++20">, DefaultIgnore, InGroup<CXXPre20Compat>;`.
  **L733 CN**: 添加一条独立语句或声明：`"C++ standards before C++20">, DefaultIgnore, InGroup<CXXPre20Compat>;`。
- **L734 EN**: Declares TableGen def record `warn_empty_init_statement`.
  **L734 CN**: 声明 TableGen def 记录 `warn_empty_init_statement`。
- **L735 EN**: Continues the surrounding expression or declaration: `"empty initialization statement of '%select{if|switch|range-based for}0' "`.
  **L735 CN**: 继续构造周围的表达式或声明：`"empty initialization statement of '%select{if|switch|range-based for}0' "`。
- **L736 EN**: Adds a standalone statement or declaration: `"has no effect">, InGroup<EmptyInitStatement>, DefaultIgnore;`.
  **L736 CN**: 添加一条独立语句或声明：`"has no effect">, InGroup<EmptyInitStatement>, DefaultIgnore;`。
- **L737 EN**: Declares TableGen def record `err_keyword_as_parameter`.
  **L737 CN**: 声明 TableGen def 记录 `err_keyword_as_parameter`。
- **L738 EN**: Adds a standalone statement or declaration: `"invalid parameter name: '%0' is a keyword">;`.
  **L738 CN**: 添加一条独立语句或声明：`"invalid parameter name: '%0' is a keyword">;`。
- **L739 EN**: Declares TableGen def record `warn_pre_cxx26_ambiguous_pack_indexing_type`.
  **L739 CN**: 声明 TableGen def 记录 `warn_pre_cxx26_ambiguous_pack_indexing_type`。
- **L740 EN**: Continues the surrounding expression or declaration: `"%0 is no longer a pack expansion but a pack "`.
  **L740 CN**: 继续构造周围的表达式或声明：`"%0 is no longer a pack expansion but a pack "`。
- **L741 EN**: Adds a standalone statement or declaration: `"indexing type; add a name to specify a pack expansion">, InGroup<CXXPre26Compat>;`.
  **L741 CN**: 添加一条独立语句或声明：`"indexing type; add a name to specify a pack expansion">, InGroup<CXXPre26Compat>;`。
- **L742 EN**: Blank line separating nearby declarations or logic blocks.
  **L742 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L743 EN**: Comment explains nearby logic, constraints, or intent: `C++ derived classes`.
  **L743 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C++ derived classes`。
- **L744 EN**: Declares TableGen def record `err_dup_virtual`.
  **L744 CN**: 声明 TableGen def 记录 `err_dup_virtual`。

### Lines 745-768

````tablegen

// C++ operator overloading
def err_literal_operator_string_prefix : Error<
  "string literal after 'operator' cannot have an encoding prefix">;
def err_literal_operator_string_not_empty : Error<
  "string literal after 'operator' must be '\"\"'">;
def warn_cxx98_compat_literal_operator : Warning<
  "literal operators are incompatible with C++98">,
  InGroup<CXX98Compat>, DefaultIgnore;

// Classes.
def err_anon_type_definition : Error<
  "declaration of anonymous %0 must be a definition">;
def err_default_delete_in_multiple_declaration : Error<
  "'= %select{default|delete}0' is a function definition and must occur in a "
  "standalone declaration">;

def warn_cxx98_compat_noexcept_expr : Warning<
  "noexcept expressions are incompatible with C++98">,
  InGroup<CXX98Compat>, DefaultIgnore;
def warn_cxx98_compat_nullptr : Warning<
  "'nullptr' is incompatible with C++98">, InGroup<CXX98Compat>, DefaultIgnore;
def ext_c_nullptr : Extension<
  "'nullptr' is a C23 extension">, InGroup<C23>;
````
- **L745 EN**: Blank line separating nearby declarations or logic blocks.
  **L745 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L746 EN**: Comment explains nearby logic, constraints, or intent: `C++ operator overloading`.
  **L746 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C++ operator overloading`。
- **L747 EN**: Declares TableGen def record `err_literal_operator_string_prefix`.
  **L747 CN**: 声明 TableGen def 记录 `err_literal_operator_string_prefix`。
- **L748 EN**: Adds a standalone statement or declaration: `"string literal after 'operator' cannot have an encoding prefix">;`.
  **L748 CN**: 添加一条独立语句或声明：`"string literal after 'operator' cannot have an encoding prefix">;`。
- **L749 EN**: Declares TableGen def record `err_literal_operator_string_not_empty`.
  **L749 CN**: 声明 TableGen def 记录 `err_literal_operator_string_not_empty`。
- **L750 EN**: Adds a standalone statement or declaration: `"string literal after 'operator' must be '\"\"'">;`.
  **L750 CN**: 添加一条独立语句或声明：`"string literal after 'operator' must be '\"\"'">;`。
- **L751 EN**: Declares TableGen def record `warn_cxx98_compat_literal_operator`.
  **L751 CN**: 声明 TableGen def 记录 `warn_cxx98_compat_literal_operator`。
- **L752 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"literal operators are incompatible with C++98">,`.
  **L752 CN**: 继续一个多行参数列表、初始化器或聚合项：`"literal operators are incompatible with C++98">,`。
- **L753 EN**: Adds a standalone statement or declaration: `InGroup<CXX98Compat>, DefaultIgnore;`.
  **L753 CN**: 添加一条独立语句或声明：`InGroup<CXX98Compat>, DefaultIgnore;`。
- **L754 EN**: Blank line separating nearby declarations or logic blocks.
  **L754 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L755 EN**: Comment explains nearby logic, constraints, or intent: `Classes.`.
  **L755 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Classes.`。
- **L756 EN**: Declares TableGen def record `err_anon_type_definition`.
  **L756 CN**: 声明 TableGen def 记录 `err_anon_type_definition`。
- **L757 EN**: Adds a standalone statement or declaration: `"declaration of anonymous %0 must be a definition">;`.
  **L757 CN**: 添加一条独立语句或声明：`"declaration of anonymous %0 must be a definition">;`。
- **L758 EN**: Declares TableGen def record `err_default_delete_in_multiple_declaration`.
  **L758 CN**: 声明 TableGen def 记录 `err_default_delete_in_multiple_declaration`。
- **L759 EN**: Continues the surrounding expression or declaration: `"'= %select{default|delete}0' is a function definition and must occur in a "`.
  **L759 CN**: 继续构造周围的表达式或声明：`"'= %select{default|delete}0' is a function definition and must occur in a "`。
- **L760 EN**: Adds a standalone statement or declaration: `"standalone declaration">;`.
  **L760 CN**: 添加一条独立语句或声明：`"standalone declaration">;`。
- **L761 EN**: Blank line separating nearby declarations or logic blocks.
  **L761 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L762 EN**: Declares TableGen def record `warn_cxx98_compat_noexcept_expr`.
  **L762 CN**: 声明 TableGen def 记录 `warn_cxx98_compat_noexcept_expr`。
- **L763 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"noexcept expressions are incompatible with C++98">,`.
  **L763 CN**: 继续一个多行参数列表、初始化器或聚合项：`"noexcept expressions are incompatible with C++98">,`。
- **L764 EN**: Adds a standalone statement or declaration: `InGroup<CXX98Compat>, DefaultIgnore;`.
  **L764 CN**: 添加一条独立语句或声明：`InGroup<CXX98Compat>, DefaultIgnore;`。
- **L765 EN**: Declares TableGen def record `warn_cxx98_compat_nullptr`.
  **L765 CN**: 声明 TableGen def 记录 `warn_cxx98_compat_nullptr`。
- **L766 EN**: Adds a standalone statement or declaration: `"'nullptr' is incompatible with C++98">, InGroup<CXX98Compat>, DefaultIgnore;`.
  **L766 CN**: 添加一条独立语句或声明：`"'nullptr' is incompatible with C++98">, InGroup<CXX98Compat>, DefaultIgnore;`。
- **L767 EN**: Declares TableGen def record `ext_c_nullptr`.
  **L767 CN**: 声明 TableGen def 记录 `ext_c_nullptr`。
- **L768 EN**: Adds a standalone statement or declaration: `"'nullptr' is a C23 extension">, InGroup<C23>;`.
  **L768 CN**: 添加一条独立语句或声明：`"'nullptr' is a C23 extension">, InGroup<C23>;`。

### Lines 769-792

````tablegen

def warn_wrong_clang_attr_namespace : Warning<
  "'__clang__' is a predefined macro name, not an attribute scope specifier; "
  "did you mean '_Clang' instead?">, InGroup<IgnoredAttributes>;
def ext_ns_enum_attribute : Extension<
  "attributes on %select{a namespace|an enumerator}0 declaration are "
  "a C++17 extension">, InGroup<CXX17>;
def warn_cxx14_compat_ns_enum_attribute : Warning<
  "attributes on %select{a namespace|an enumerator}0 declaration are "
  "incompatible with C++ standards before C++17">,
  InGroup<CXXPre17CompatPedantic>, DefaultIgnore;
def warn_cxx98_compat_alignas : Warning<"'alignas' is incompatible with C++98">,
  InGroup<CXX98Compat>, DefaultIgnore;
def warn_cxx98_compat_attribute : Warning<
  "[[]] attributes are incompatible with C++ standards before C++11">,
  InGroup<CXX98Compat>, DefaultIgnore;
def warn_ext_cxx11_attributes : Extension<
  "[[]] attributes are a C++11 extension">,
  InGroup<CXX11>;
def warn_pre_c23_compat_attributes : Warning<
  "[[]] attributes are incompatible with C standards before C23">,
  DefaultIgnore, InGroup<CPre23Compat>;
def warn_ext_c23_attributes : Extension<
  "[[]] attributes are a C23 extension">,
````
- **L769 EN**: Blank line separating nearby declarations or logic blocks.
  **L769 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L770 EN**: Declares TableGen def record `warn_wrong_clang_attr_namespace`.
  **L770 CN**: 声明 TableGen def 记录 `warn_wrong_clang_attr_namespace`。
- **L771 EN**: Continues the surrounding expression or declaration: `"'__clang__' is a predefined macro name, not an attribute scope specifier; "`.
  **L771 CN**: 继续构造周围的表达式或声明：`"'__clang__' is a predefined macro name, not an attribute scope specifier; "`。
- **L772 EN**: Adds a standalone statement or declaration: `"did you mean '_Clang' instead?">, InGroup<IgnoredAttributes>;`.
  **L772 CN**: 添加一条独立语句或声明：`"did you mean '_Clang' instead?">, InGroup<IgnoredAttributes>;`。
- **L773 EN**: Declares TableGen def record `ext_ns_enum_attribute`.
  **L773 CN**: 声明 TableGen def 记录 `ext_ns_enum_attribute`。
- **L774 EN**: Continues the surrounding expression or declaration: `"attributes on %select{a namespace|an enumerator}0 declaration are "`.
  **L774 CN**: 继续构造周围的表达式或声明：`"attributes on %select{a namespace|an enumerator}0 declaration are "`。
- **L775 EN**: Adds a standalone statement or declaration: `"a C++17 extension">, InGroup<CXX17>;`.
  **L775 CN**: 添加一条独立语句或声明：`"a C++17 extension">, InGroup<CXX17>;`。
- **L776 EN**: Declares TableGen def record `warn_cxx14_compat_ns_enum_attribute`.
  **L776 CN**: 声明 TableGen def 记录 `warn_cxx14_compat_ns_enum_attribute`。
- **L777 EN**: Continues the surrounding expression or declaration: `"attributes on %select{a namespace|an enumerator}0 declaration are "`.
  **L777 CN**: 继续构造周围的表达式或声明：`"attributes on %select{a namespace|an enumerator}0 declaration are "`。
- **L778 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"incompatible with C++ standards before C++17">,`.
  **L778 CN**: 继续一个多行参数列表、初始化器或聚合项：`"incompatible with C++ standards before C++17">,`。
- **L779 EN**: Adds a standalone statement or declaration: `InGroup<CXXPre17CompatPedantic>, DefaultIgnore;`.
  **L779 CN**: 添加一条独立语句或声明：`InGroup<CXXPre17CompatPedantic>, DefaultIgnore;`。
- **L780 EN**: Declares TableGen def record `warn_cxx98_compat_alignas`.
  **L780 CN**: 声明 TableGen def 记录 `warn_cxx98_compat_alignas`。
- **L781 EN**: Adds a standalone statement or declaration: `InGroup<CXX98Compat>, DefaultIgnore;`.
  **L781 CN**: 添加一条独立语句或声明：`InGroup<CXX98Compat>, DefaultIgnore;`。
- **L782 EN**: Declares TableGen def record `warn_cxx98_compat_attribute`.
  **L782 CN**: 声明 TableGen def 记录 `warn_cxx98_compat_attribute`。
- **L783 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"[[]] attributes are incompatible with C++ standards before C++11">,`.
  **L783 CN**: 继续一个多行参数列表、初始化器或聚合项：`"[[]] attributes are incompatible with C++ standards before C++11">,`。
- **L784 EN**: Adds a standalone statement or declaration: `InGroup<CXX98Compat>, DefaultIgnore;`.
  **L784 CN**: 添加一条独立语句或声明：`InGroup<CXX98Compat>, DefaultIgnore;`。
- **L785 EN**: Declares TableGen def record `warn_ext_cxx11_attributes`.
  **L785 CN**: 声明 TableGen def 记录 `warn_ext_cxx11_attributes`。
- **L786 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"[[]] attributes are a C++11 extension">,`.
  **L786 CN**: 继续一个多行参数列表、初始化器或聚合项：`"[[]] attributes are a C++11 extension">,`。
- **L787 EN**: Adds a standalone statement or declaration: `InGroup<CXX11>;`.
  **L787 CN**: 添加一条独立语句或声明：`InGroup<CXX11>;`。
- **L788 EN**: Declares TableGen def record `warn_pre_c23_compat_attributes`.
  **L788 CN**: 声明 TableGen def 记录 `warn_pre_c23_compat_attributes`。
- **L789 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"[[]] attributes are incompatible with C standards before C23">,`.
  **L789 CN**: 继续一个多行参数列表、初始化器或聚合项：`"[[]] attributes are incompatible with C standards before C23">,`。
- **L790 EN**: Adds a standalone statement or declaration: `DefaultIgnore, InGroup<CPre23Compat>;`.
  **L790 CN**: 添加一条独立语句或声明：`DefaultIgnore, InGroup<CPre23Compat>;`。
- **L791 EN**: Declares TableGen def record `warn_ext_c23_attributes`.
  **L791 CN**: 声明 TableGen def 记录 `warn_ext_c23_attributes`。
- **L792 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"[[]] attributes are a C23 extension">,`.
  **L792 CN**: 继续一个多行参数列表、初始化器或聚合项：`"[[]] attributes are a C23 extension">,`。

### Lines 793-816

````tablegen
  InGroup<C23>;
def err_cxx11_attribute_forbids_arguments : Error<
  "attribute %0 cannot have an argument list">;
def err_attribute_requires_arguments : Error<
  "parentheses must be omitted if %0 attribute's argument list is empty">;
def err_cxx11_attribute_forbids_ellipsis : Error<
  "attribute %0 cannot be used as an attribute pack">;
def warn_cxx14_compat_using_attribute_ns : Warning<
  "default scope specifier for attributes is incompatible with C++ standards "
  "before C++17">, InGroup<CXXPre17Compat>, DefaultIgnore;
def ext_using_attribute_ns : ExtWarn<
  "default scope specifier for attributes is a C++17 extension">,
  InGroup<CXX17>;
def err_using_attribute_ns_conflict : Error<
  "attribute with scope specifier cannot follow default scope specifier">;
def err_attributes_not_allowed : Error<"an attribute list cannot appear here">;
def err_keyword_not_allowed : Error<"%0 cannot appear here">;
def ext_cxx11_attr_placement : ExtWarn<
  "ISO C++ does not allow %select{an attribute list|%0}1 to appear here">,
  InGroup<DiagGroup<"cxx-attribute-extension">>;
def err_attributes_misplaced : Error<"misplaced attributes; expected attributes here">;
def err_keyword_misplaced : Error<"misplaced %0; expected %0 here">;
def err_l_square_l_square_not_attribute : Error<
  "C++11 only allows consecutive left square brackets when "
````
- **L793 EN**: Adds a standalone statement or declaration: `InGroup<C23>;`.
  **L793 CN**: 添加一条独立语句或声明：`InGroup<C23>;`。
- **L794 EN**: Declares TableGen def record `err_cxx11_attribute_forbids_arguments`.
  **L794 CN**: 声明 TableGen def 记录 `err_cxx11_attribute_forbids_arguments`。
- **L795 EN**: Adds a standalone statement or declaration: `"attribute %0 cannot have an argument list">;`.
  **L795 CN**: 添加一条独立语句或声明：`"attribute %0 cannot have an argument list">;`。
- **L796 EN**: Declares TableGen def record `err_attribute_requires_arguments`.
  **L796 CN**: 声明 TableGen def 记录 `err_attribute_requires_arguments`。
- **L797 EN**: Adds a standalone statement or declaration: `"parentheses must be omitted if %0 attribute's argument list is empty">;`.
  **L797 CN**: 添加一条独立语句或声明：`"parentheses must be omitted if %0 attribute's argument list is empty">;`。
- **L798 EN**: Declares TableGen def record `err_cxx11_attribute_forbids_ellipsis`.
  **L798 CN**: 声明 TableGen def 记录 `err_cxx11_attribute_forbids_ellipsis`。
- **L799 EN**: Adds a standalone statement or declaration: `"attribute %0 cannot be used as an attribute pack">;`.
  **L799 CN**: 添加一条独立语句或声明：`"attribute %0 cannot be used as an attribute pack">;`。
- **L800 EN**: Declares TableGen def record `warn_cxx14_compat_using_attribute_ns`.
  **L800 CN**: 声明 TableGen def 记录 `warn_cxx14_compat_using_attribute_ns`。
- **L801 EN**: Continues the surrounding expression or declaration: `"default scope specifier for attributes is incompatible with C++ standards "`.
  **L801 CN**: 继续构造周围的表达式或声明：`"default scope specifier for attributes is incompatible with C++ standards "`。
- **L802 EN**: Adds a standalone statement or declaration: `"before C++17">, InGroup<CXXPre17Compat>, DefaultIgnore;`.
  **L802 CN**: 添加一条独立语句或声明：`"before C++17">, InGroup<CXXPre17Compat>, DefaultIgnore;`。
- **L803 EN**: Declares TableGen def record `ext_using_attribute_ns`.
  **L803 CN**: 声明 TableGen def 记录 `ext_using_attribute_ns`。
- **L804 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"default scope specifier for attributes is a C++17 extension">,`.
  **L804 CN**: 继续一个多行参数列表、初始化器或聚合项：`"default scope specifier for attributes is a C++17 extension">,`。
- **L805 EN**: Adds a standalone statement or declaration: `InGroup<CXX17>;`.
  **L805 CN**: 添加一条独立语句或声明：`InGroup<CXX17>;`。
- **L806 EN**: Declares TableGen def record `err_using_attribute_ns_conflict`.
  **L806 CN**: 声明 TableGen def 记录 `err_using_attribute_ns_conflict`。
- **L807 EN**: Adds a standalone statement or declaration: `"attribute with scope specifier cannot follow default scope specifier">;`.
  **L807 CN**: 添加一条独立语句或声明：`"attribute with scope specifier cannot follow default scope specifier">;`。
- **L808 EN**: Declares TableGen def record `err_attributes_not_allowed`.
  **L808 CN**: 声明 TableGen def 记录 `err_attributes_not_allowed`。
- **L809 EN**: Declares TableGen def record `err_keyword_not_allowed`.
  **L809 CN**: 声明 TableGen def 记录 `err_keyword_not_allowed`。
- **L810 EN**: Declares TableGen def record `ext_cxx11_attr_placement`.
  **L810 CN**: 声明 TableGen def 记录 `ext_cxx11_attr_placement`。
- **L811 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"ISO C++ does not allow %select{an attribute list|%0}1 to appear here">,`.
  **L811 CN**: 继续一个多行参数列表、初始化器或聚合项：`"ISO C++ does not allow %select{an attribute list|%0}1 to appear here">,`。
- **L812 EN**: Adds a standalone statement or declaration: `InGroup<DiagGroup<"cxx-attribute-extension">>;`.
  **L812 CN**: 添加一条独立语句或声明：`InGroup<DiagGroup<"cxx-attribute-extension">>;`。
- **L813 EN**: Declares TableGen def record `err_attributes_misplaced`.
  **L813 CN**: 声明 TableGen def 记录 `err_attributes_misplaced`。
- **L814 EN**: Declares TableGen def record `err_keyword_misplaced`.
  **L814 CN**: 声明 TableGen def 记录 `err_keyword_misplaced`。
- **L815 EN**: Declares TableGen def record `err_l_square_l_square_not_attribute`.
  **L815 CN**: 声明 TableGen def 记录 `err_l_square_l_square_not_attribute`。
- **L816 EN**: Continues the surrounding expression or declaration: `"C++11 only allows consecutive left square brackets when "`.
  **L816 CN**: 继续构造周围的表达式或声明：`"C++11 only allows consecutive left square brackets when "`。

### Lines 817-840

````tablegen
  "introducing an attribute">;
def err_attribute_argument_parm_pack_not_supported : Error<
  "attribute %0 does not support argument pack expansion">;
def err_ms_declspec_type : Error<
  "__declspec attributes must be an identifier or string literal">;
def err_ms_property_no_getter_or_putter : Error<
  "property does not specify a getter or a putter">;
def err_ms_property_unknown_accessor : Error<
  "expected 'get' or 'put' in property declaration">;
def err_ms_property_has_set_accessor : Error<
  "putter for property must be specified as 'put', not 'set'">;
def err_ms_property_missing_accessor_kind : Error<
  "missing 'get=' or 'put='">;
def err_ms_property_expected_equal : Error<
  "expected '=' after '%0'">;
def err_ms_property_duplicate_accessor : Error<
  "property declaration specifies '%0' accessor twice">;
def err_ms_property_expected_accessor_name : Error<
  "expected name of accessor method">;
def err_ms_property_expected_comma_or_rparen : Error<
  "expected ',' or ')' at end of property accessor list">;
def err_ms_property_initializer : Error<
  "property declaration cannot have a default member initializer">;
def ext_invalid_attribute_argument
````
- **L817 EN**: Adds a standalone statement or declaration: `"introducing an attribute">;`.
  **L817 CN**: 添加一条独立语句或声明：`"introducing an attribute">;`。
- **L818 EN**: Declares TableGen def record `err_attribute_argument_parm_pack_not_supported`.
  **L818 CN**: 声明 TableGen def 记录 `err_attribute_argument_parm_pack_not_supported`。
- **L819 EN**: Adds a standalone statement or declaration: `"attribute %0 does not support argument pack expansion">;`.
  **L819 CN**: 添加一条独立语句或声明：`"attribute %0 does not support argument pack expansion">;`。
- **L820 EN**: Declares TableGen def record `err_ms_declspec_type`.
  **L820 CN**: 声明 TableGen def 记录 `err_ms_declspec_type`。
- **L821 EN**: Applies compiler-specific attributes that refine ABI, layout, inlining, or diagnostics: `"__declspec attributes must be an identifier or string literal">;`.
  **L821 CN**: 应用编译器特定属性，以细化 ABI、布局、内联或诊断行为：`"__declspec attributes must be an identifier or string literal">;`。
- **L822 EN**: Declares TableGen def record `err_ms_property_no_getter_or_putter`.
  **L822 CN**: 声明 TableGen def 记录 `err_ms_property_no_getter_or_putter`。
- **L823 EN**: Adds a standalone statement or declaration: `"property does not specify a getter or a putter">;`.
  **L823 CN**: 添加一条独立语句或声明：`"property does not specify a getter or a putter">;`。
- **L824 EN**: Declares TableGen def record `err_ms_property_unknown_accessor`.
  **L824 CN**: 声明 TableGen def 记录 `err_ms_property_unknown_accessor`。
- **L825 EN**: Adds a standalone statement or declaration: `"expected 'get' or 'put' in property declaration">;`.
  **L825 CN**: 添加一条独立语句或声明：`"expected 'get' or 'put' in property declaration">;`。
- **L826 EN**: Declares TableGen def record `err_ms_property_has_set_accessor`.
  **L826 CN**: 声明 TableGen def 记录 `err_ms_property_has_set_accessor`。
- **L827 EN**: Adds a standalone statement or declaration: `"putter for property must be specified as 'put', not 'set'">;`.
  **L827 CN**: 添加一条独立语句或声明：`"putter for property must be specified as 'put', not 'set'">;`。
- **L828 EN**: Declares TableGen def record `err_ms_property_missing_accessor_kind`.
  **L828 CN**: 声明 TableGen def 记录 `err_ms_property_missing_accessor_kind`。
- **L829 EN**: Adds a standalone statement or declaration: `"missing 'get=' or 'put='">;`.
  **L829 CN**: 添加一条独立语句或声明：`"missing 'get=' or 'put='">;`。
- **L830 EN**: Declares TableGen def record `err_ms_property_expected_equal`.
  **L830 CN**: 声明 TableGen def 记录 `err_ms_property_expected_equal`。
- **L831 EN**: Adds a standalone statement or declaration: `"expected '=' after '%0'">;`.
  **L831 CN**: 添加一条独立语句或声明：`"expected '=' after '%0'">;`。
- **L832 EN**: Declares TableGen def record `err_ms_property_duplicate_accessor`.
  **L832 CN**: 声明 TableGen def 记录 `err_ms_property_duplicate_accessor`。
- **L833 EN**: Adds a standalone statement or declaration: `"property declaration specifies '%0' accessor twice">;`.
  **L833 CN**: 添加一条独立语句或声明：`"property declaration specifies '%0' accessor twice">;`。
- **L834 EN**: Declares TableGen def record `err_ms_property_expected_accessor_name`.
  **L834 CN**: 声明 TableGen def 记录 `err_ms_property_expected_accessor_name`。
- **L835 EN**: Adds a standalone statement or declaration: `"expected name of accessor method">;`.
  **L835 CN**: 添加一条独立语句或声明：`"expected name of accessor method">;`。
- **L836 EN**: Declares TableGen def record `err_ms_property_expected_comma_or_rparen`.
  **L836 CN**: 声明 TableGen def 记录 `err_ms_property_expected_comma_or_rparen`。
- **L837 EN**: Adds a standalone statement or declaration: `"expected ',' or ')' at end of property accessor list">;`.
  **L837 CN**: 添加一条独立语句或声明：`"expected ',' or ')' at end of property accessor list">;`。
- **L838 EN**: Declares TableGen def record `err_ms_property_initializer`.
  **L838 CN**: 声明 TableGen def 记录 `err_ms_property_initializer`。
- **L839 EN**: Adds a standalone statement or declaration: `"property declaration cannot have a default member initializer">;`.
  **L839 CN**: 添加一条独立语句或声明：`"property declaration cannot have a default member initializer">;`。
- **L840 EN**: Declares TableGen def record `ext_invalid_attribute_argument`.
  **L840 CN**: 声明 TableGen def 记录 `ext_invalid_attribute_argument`。

### Lines 841-864

````tablegen
    : Extension<"'%0' is not allowed in an attribute argument list">,
    InGroup<DiagGroup<"attribute-preprocessor-tokens">>;

def err_assume_attr_expects_cond_expr : Error<
  "use of this expression in an %0 attribute requires parentheses">;

def warn_cxx20_compat_explicit_bool : Warning<
  "this expression will be parsed as explicit(bool) in C++20">,
  InGroup<CXX20Compat>, DefaultIgnore;
def warn_cxx17_compat_explicit_bool : Warning<
  "explicit(bool) is incompatible with C++ standards before C++20">,
  InGroup<CXXPre20Compat>, DefaultIgnore;
def ext_explicit_bool : ExtWarn<"explicit(bool) is a C++20 extension">,
  InGroup<CXX20>;

/// C++ Templates
def err_expected_template : Error<"expected template">;
def err_unknown_template_name : Error<
  "unknown template name %0">;
def err_expected_comma_greater : Error<
  "expected ',' or '>' in template-parameter-list">;
def err_class_on_template_template_param
    : Error<"template template parameter requires 'class'%select{| or "
            "'typename'}0 after the parameter list">;
````
- **L841 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Extension<"'%0' is not allowed in an attribute argument list">,`.
  **L841 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Extension<"'%0' is not allowed in an attribute argument list">,`。
- **L842 EN**: Adds a standalone statement or declaration: `InGroup<DiagGroup<"attribute-preprocessor-tokens">>;`.
  **L842 CN**: 添加一条独立语句或声明：`InGroup<DiagGroup<"attribute-preprocessor-tokens">>;`。
- **L843 EN**: Blank line separating nearby declarations or logic blocks.
  **L843 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L844 EN**: Declares TableGen def record `err_assume_attr_expects_cond_expr`.
  **L844 CN**: 声明 TableGen def 记录 `err_assume_attr_expects_cond_expr`。
- **L845 EN**: Adds a standalone statement or declaration: `"use of this expression in an %0 attribute requires parentheses">;`.
  **L845 CN**: 添加一条独立语句或声明：`"use of this expression in an %0 attribute requires parentheses">;`。
- **L846 EN**: Blank line separating nearby declarations or logic blocks.
  **L846 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L847 EN**: Declares TableGen def record `warn_cxx20_compat_explicit_bool`.
  **L847 CN**: 声明 TableGen def 记录 `warn_cxx20_compat_explicit_bool`。
- **L848 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"this expression will be parsed as explicit(bool) in C++20">,`.
  **L848 CN**: 继续一个多行参数列表、初始化器或聚合项：`"this expression will be parsed as explicit(bool) in C++20">,`。
- **L849 EN**: Adds a standalone statement or declaration: `InGroup<CXX20Compat>, DefaultIgnore;`.
  **L849 CN**: 添加一条独立语句或声明：`InGroup<CXX20Compat>, DefaultIgnore;`。
- **L850 EN**: Declares TableGen def record `warn_cxx17_compat_explicit_bool`.
  **L850 CN**: 声明 TableGen def 记录 `warn_cxx17_compat_explicit_bool`。
- **L851 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"explicit(bool) is incompatible with C++ standards before C++20">,`.
  **L851 CN**: 继续一个多行参数列表、初始化器或聚合项：`"explicit(bool) is incompatible with C++ standards before C++20">,`。
- **L852 EN**: Adds a standalone statement or declaration: `InGroup<CXXPre20Compat>, DefaultIgnore;`.
  **L852 CN**: 添加一条独立语句或声明：`InGroup<CXXPre20Compat>, DefaultIgnore;`。
- **L853 EN**: Declares TableGen def record `ext_explicit_bool`.
  **L853 CN**: 声明 TableGen def 记录 `ext_explicit_bool`。
- **L854 EN**: Adds a standalone statement or declaration: `InGroup<CXX20>;`.
  **L854 CN**: 添加一条独立语句或声明：`InGroup<CXX20>;`。
- **L855 EN**: Blank line separating nearby declarations or logic blocks.
  **L855 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L856 EN**: Comment explains nearby logic, constraints, or intent: `C++ Templates`.
  **L856 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C++ Templates`。
- **L857 EN**: Declares TableGen def record `err_expected_template`.
  **L857 CN**: 声明 TableGen def 记录 `err_expected_template`。
- **L858 EN**: Declares TableGen def record `err_unknown_template_name`.
  **L858 CN**: 声明 TableGen def 记录 `err_unknown_template_name`。
- **L859 EN**: Adds a standalone statement or declaration: `"unknown template name %0">;`.
  **L859 CN**: 添加一条独立语句或声明：`"unknown template name %0">;`。
- **L860 EN**: Declares TableGen def record `err_expected_comma_greater`.
  **L860 CN**: 声明 TableGen def 记录 `err_expected_comma_greater`。
- **L861 EN**: Adds a standalone statement or declaration: `"expected ',' or '>' in template-parameter-list">;`.
  **L861 CN**: 添加一条独立语句或声明：`"expected ',' or '>' in template-parameter-list">;`。
- **L862 EN**: Declares TableGen def record `err_class_on_template_template_param`.
  **L862 CN**: 声明 TableGen def 记录 `err_class_on_template_template_param`。
- **L863 EN**: Continues the surrounding expression or declaration: `: Error<"template template parameter requires 'class'%select{| or "`.
  **L863 CN**: 继续构造周围的表达式或声明：`: Error<"template template parameter requires 'class'%select{| or "`。
- **L864 EN**: Adds a standalone statement or declaration: `"'typename'}0 after the parameter list">;`.
  **L864 CN**: 添加一条独立语句或声明：`"'typename'}0 after the parameter list">;`。

### Lines 865-888

````tablegen
def ext_template_template_param_typename : ExtWarn<
  "template template parameter using 'typename' is a C++17 extension">,
  InGroup<CXX17>;
def warn_cxx14_compat_template_template_param_typename : Warning<
  "template template parameter using 'typename' is "
  "incompatible with C++ standards before C++17">,
  InGroup<CXXPre17Compat>, DefaultIgnore;
def err_template_spec_syntax_non_template : Error<
  "identifier followed by '<' indicates a class template specialization but "
  "%0 %select{does not refer to a template|refers to a function template|"
  "<unused>|refers to a variable template|<unused>|<unused>|"
  "refers to a concept}1">;
def err_id_after_template_in_nested_name_spec : Error<
  "expected template name after 'template' keyword in nested name specifier">;
def err_unexpected_template_in_unqualified_id : Error<
  "'template' keyword not permitted here">;
def err_unexpected_template_in_destructor_name : Error<
  "'template' keyword not permitted in destructor name">;
def err_unexpected_template_after_using : Error<
  "'template' keyword not permitted after 'using' keyword">;
def err_two_right_angle_brackets_need_space : Error<
  "a space is required between consecutive right angle brackets (use '> >')">;
def err_right_angle_bracket_equal_needs_space : Error<
  "a space is required between a right angle bracket and an equals sign "
````
- **L865 EN**: Declares TableGen def record `ext_template_template_param_typename`.
  **L865 CN**: 声明 TableGen def 记录 `ext_template_template_param_typename`。
- **L866 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"template template parameter using 'typename' is a C++17 extension">,`.
  **L866 CN**: 继续一个多行参数列表、初始化器或聚合项：`"template template parameter using 'typename' is a C++17 extension">,`。
- **L867 EN**: Adds a standalone statement or declaration: `InGroup<CXX17>;`.
  **L867 CN**: 添加一条独立语句或声明：`InGroup<CXX17>;`。
- **L868 EN**: Declares TableGen def record `warn_cxx14_compat_template_template_param_typename`.
  **L868 CN**: 声明 TableGen def 记录 `warn_cxx14_compat_template_template_param_typename`。
- **L869 EN**: Continues the surrounding expression or declaration: `"template template parameter using 'typename' is "`.
  **L869 CN**: 继续构造周围的表达式或声明：`"template template parameter using 'typename' is "`。
- **L870 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"incompatible with C++ standards before C++17">,`.
  **L870 CN**: 继续一个多行参数列表、初始化器或聚合项：`"incompatible with C++ standards before C++17">,`。
- **L871 EN**: Adds a standalone statement or declaration: `InGroup<CXXPre17Compat>, DefaultIgnore;`.
  **L871 CN**: 添加一条独立语句或声明：`InGroup<CXXPre17Compat>, DefaultIgnore;`。
- **L872 EN**: Declares TableGen def record `err_template_spec_syntax_non_template`.
  **L872 CN**: 声明 TableGen def 记录 `err_template_spec_syntax_non_template`。
- **L873 EN**: Continues the surrounding expression or declaration: `"identifier followed by '<' indicates a class template specialization but "`.
  **L873 CN**: 继续构造周围的表达式或声明：`"identifier followed by '<' indicates a class template specialization but "`。
- **L874 EN**: Continues the surrounding expression or declaration: `"%0 %select{does not refer to a template|refers to a function template|"`.
  **L874 CN**: 继续构造周围的表达式或声明：`"%0 %select{does not refer to a template|refers to a function template|"`。
- **L875 EN**: Continues the surrounding expression or declaration: `"<unused>|refers to a variable template|<unused>|<unused>|"`.
  **L875 CN**: 继续构造周围的表达式或声明：`"<unused>|refers to a variable template|<unused>|<unused>|"`。
- **L876 EN**: Adds a standalone statement or declaration: `"refers to a concept}1">;`.
  **L876 CN**: 添加一条独立语句或声明：`"refers to a concept}1">;`。
- **L877 EN**: Declares TableGen def record `err_id_after_template_in_nested_name_spec`.
  **L877 CN**: 声明 TableGen def 记录 `err_id_after_template_in_nested_name_spec`。
- **L878 EN**: Adds a standalone statement or declaration: `"expected template name after 'template' keyword in nested name specifier">;`.
  **L878 CN**: 添加一条独立语句或声明：`"expected template name after 'template' keyword in nested name specifier">;`。
- **L879 EN**: Declares TableGen def record `err_unexpected_template_in_unqualified_id`.
  **L879 CN**: 声明 TableGen def 记录 `err_unexpected_template_in_unqualified_id`。
- **L880 EN**: Adds a standalone statement or declaration: `"'template' keyword not permitted here">;`.
  **L880 CN**: 添加一条独立语句或声明：`"'template' keyword not permitted here">;`。
- **L881 EN**: Declares TableGen def record `err_unexpected_template_in_destructor_name`.
  **L881 CN**: 声明 TableGen def 记录 `err_unexpected_template_in_destructor_name`。
- **L882 EN**: Adds a standalone statement or declaration: `"'template' keyword not permitted in destructor name">;`.
  **L882 CN**: 添加一条独立语句或声明：`"'template' keyword not permitted in destructor name">;`。
- **L883 EN**: Declares TableGen def record `err_unexpected_template_after_using`.
  **L883 CN**: 声明 TableGen def 记录 `err_unexpected_template_after_using`。
- **L884 EN**: Adds a standalone statement or declaration: `"'template' keyword not permitted after 'using' keyword">;`.
  **L884 CN**: 添加一条独立语句或声明：`"'template' keyword not permitted after 'using' keyword">;`。
- **L885 EN**: Declares TableGen def record `err_two_right_angle_brackets_need_space`.
  **L885 CN**: 声明 TableGen def 记录 `err_two_right_angle_brackets_need_space`。
- **L886 EN**: Executes a call or declaration centered on `brackets`.
  **L886 CN**: 执行以 `brackets` 为核心的调用或声明。
- **L887 EN**: Declares TableGen def record `err_right_angle_bracket_equal_needs_space`.
  **L887 CN**: 声明 TableGen def 记录 `err_right_angle_bracket_equal_needs_space`。
- **L888 EN**: Continues the surrounding expression or declaration: `"a space is required between a right angle bracket and an equals sign "`.
  **L888 CN**: 继续构造周围的表达式或声明：`"a space is required between a right angle bracket and an equals sign "`。

### Lines 889-912

````tablegen
  "(use '> =')">;
def warn_cxx11_right_shift_in_template_arg : Warning<
  "use of right-shift operator ('>>') in template argument will require "
  "parentheses in C++11">, InGroup<CXX11Compat>;
def warn_cxx98_compat_two_right_angle_brackets : Warning<
  "consecutive right angle brackets are incompatible with C++98 (use '> >')">,
  InGroup<CXX98Compat>, DefaultIgnore;
def err_templated_invalid_declaration : Error<
  "a static_assert declaration cannot be a template">;
def err_multiple_template_declarators : Error<
  "%select{|a template declaration|an explicit template specialization|"
  "an explicit template instantiation}0 can "
  "only %select{|declare|declare|instantiate}0 a single entity">;
def err_explicit_instantiation_with_definition : Error<
  "explicit template instantiation cannot have a definition; if this "
  "definition is meant to be an explicit specialization, add '<>' after the "
  "'template' keyword">;
def err_template_defn_explicit_instantiation : Error<
  "%select{function|class|variable}0 cannot be defined in an explicit instantiation; if this "
  "declaration is meant to be a %select{function|class|variable}0 definition, remove the 'template' keyword">;
def err_friend_explicit_instantiation : Error<
  "friend cannot be declared in an explicit instantiation; if this "
  "declaration is meant to be a friend declaration, remove the 'template' keyword">;
def err_explicit_instantiation_enum : Error<
````
- **L889 EN**: Executes a call or declaration centered on `"`.
  **L889 CN**: 执行以 `"` 为核心的调用或声明。
- **L890 EN**: Declares TableGen def record `warn_cxx11_right_shift_in_template_arg`.
  **L890 CN**: 声明 TableGen def 记录 `warn_cxx11_right_shift_in_template_arg`。
- **L891 EN**: Continues logic associated with callable symbol `operator`.
  **L891 CN**: 继续与可调用符号 `operator` 相关的逻辑。
- **L892 EN**: Adds a standalone statement or declaration: `"parentheses in C++11">, InGroup<CXX11Compat>;`.
  **L892 CN**: 添加一条独立语句或声明：`"parentheses in C++11">, InGroup<CXX11Compat>;`。
- **L893 EN**: Declares TableGen def record `warn_cxx98_compat_two_right_angle_brackets`.
  **L893 CN**: 声明 TableGen def 记录 `warn_cxx98_compat_two_right_angle_brackets`。
- **L894 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"consecutive right angle brackets are incompatible with C++98 (use '> >')">,`.
  **L894 CN**: 继续一个多行参数列表、初始化器或聚合项：`"consecutive right angle brackets are incompatible with C++98 (use '> >')">,`。
- **L895 EN**: Adds a standalone statement or declaration: `InGroup<CXX98Compat>, DefaultIgnore;`.
  **L895 CN**: 添加一条独立语句或声明：`InGroup<CXX98Compat>, DefaultIgnore;`。
- **L896 EN**: Declares TableGen def record `err_templated_invalid_declaration`.
  **L896 CN**: 声明 TableGen def 记录 `err_templated_invalid_declaration`。
- **L897 EN**: Adds a standalone statement or declaration: `"a static_assert declaration cannot be a template">;`.
  **L897 CN**: 添加一条独立语句或声明：`"a static_assert declaration cannot be a template">;`。
- **L898 EN**: Declares TableGen def record `err_multiple_template_declarators`.
  **L898 CN**: 声明 TableGen def 记录 `err_multiple_template_declarators`。
- **L899 EN**: Continues the surrounding expression or declaration: `"%select{|a template declaration|an explicit template specialization|"`.
  **L899 CN**: 继续构造周围的表达式或声明：`"%select{|a template declaration|an explicit template specialization|"`。
- **L900 EN**: Continues the surrounding expression or declaration: `"an explicit template instantiation}0 can "`.
  **L900 CN**: 继续构造周围的表达式或声明：`"an explicit template instantiation}0 can "`。
- **L901 EN**: Adds a standalone statement or declaration: `"only %select{|declare|declare|instantiate}0 a single entity">;`.
  **L901 CN**: 添加一条独立语句或声明：`"only %select{|declare|declare|instantiate}0 a single entity">;`。
- **L902 EN**: Declares TableGen def record `err_explicit_instantiation_with_definition`.
  **L902 CN**: 声明 TableGen def 记录 `err_explicit_instantiation_with_definition`。
- **L903 EN**: Continues the surrounding expression or declaration: `"explicit template instantiation cannot have a definition; if this "`.
  **L903 CN**: 继续构造周围的表达式或声明：`"explicit template instantiation cannot have a definition; if this "`。
- **L904 EN**: Continues the surrounding expression or declaration: `"definition is meant to be an explicit specialization, add '<>' after the "`.
  **L904 CN**: 继续构造周围的表达式或声明：`"definition is meant to be an explicit specialization, add '<>' after the "`。
- **L905 EN**: Adds a standalone statement or declaration: `"'template' keyword">;`.
  **L905 CN**: 添加一条独立语句或声明：`"'template' keyword">;`。
- **L906 EN**: Declares TableGen def record `err_template_defn_explicit_instantiation`.
  **L906 CN**: 声明 TableGen def 记录 `err_template_defn_explicit_instantiation`。
- **L907 EN**: Continues the surrounding expression or declaration: `"%select{function|class|variable}0 cannot be defined in an explicit instantiation; if this "`.
  **L907 CN**: 继续构造周围的表达式或声明：`"%select{function|class|variable}0 cannot be defined in an explicit instantiation; if this "`。
- **L908 EN**: Adds a standalone statement or declaration: `"declaration is meant to be a %select{function|class|variable}0 definition, remove the 'template' keyword">;`.
  **L908 CN**: 添加一条独立语句或声明：`"declaration is meant to be a %select{function|class|variable}0 definition, remove the 'template' keyword">;`。
- **L909 EN**: Declares TableGen def record `err_friend_explicit_instantiation`.
  **L909 CN**: 声明 TableGen def 记录 `err_friend_explicit_instantiation`。
- **L910 EN**: Continues the surrounding expression or declaration: `"friend cannot be declared in an explicit instantiation; if this "`.
  **L910 CN**: 继续构造周围的表达式或声明：`"friend cannot be declared in an explicit instantiation; if this "`。
- **L911 EN**: Adds a standalone statement or declaration: `"declaration is meant to be a friend declaration, remove the 'template' keyword">;`.
  **L911 CN**: 添加一条独立语句或声明：`"declaration is meant to be a friend declaration, remove the 'template' keyword">;`。
- **L912 EN**: Declares TableGen def record `err_explicit_instantiation_enum`.
  **L912 CN**: 声明 TableGen def 记录 `err_explicit_instantiation_enum`。

### Lines 913-936

````tablegen
  "enumerations cannot be explicitly instantiated">;
def err_expected_template_parameter : Error<"expected template parameter">;
def err_empty_requires_expr : Error<
  "a requires expression must contain at least one requirement">;
def err_requires_expr_parameter_list_ellipsis : Error<
  "varargs not allowed in requires expression">;
def err_requires_expr_explicit_object_parameter: Error<
  "a requires expression cannot have an explicit object parameter">;
def err_expected_semi_requirement : Error<
  "expected ';' at end of requirement">;
def err_requires_expr_missing_arrow : Error<
  "expected '->' before expression type requirement">;
def err_requires_expr_expected_type_constraint : Error<
  "expected concept name with optional arguments">;
def err_requires_expr_simple_requirement_noexcept : Error<
  "'noexcept' can only be used in a compound requirement (with '{' '}' around "
  "the expression)">;
def err_requires_expr_in_simple_requirement : Error<
  "requires expression in requirement body; did "
  "you intend to place it in a nested requirement? (add another 'requires' "
  "before the expression)">;
def missing_template_arg_list_after_template_kw : Extension<
  "a template argument list is expected after a name prefixed by the template "
  "keyword">, InGroup<DiagGroup<"missing-template-arg-list-after-template-kw">>,
````
- **L913 EN**: Adds a standalone statement or declaration: `"enumerations cannot be explicitly instantiated">;`.
  **L913 CN**: 添加一条独立语句或声明：`"enumerations cannot be explicitly instantiated">;`。
- **L914 EN**: Declares TableGen def record `err_expected_template_parameter`.
  **L914 CN**: 声明 TableGen def 记录 `err_expected_template_parameter`。
- **L915 EN**: Declares TableGen def record `err_empty_requires_expr`.
  **L915 CN**: 声明 TableGen def 记录 `err_empty_requires_expr`。
- **L916 EN**: Adds a standalone statement or declaration: `"a requires expression must contain at least one requirement">;`.
  **L916 CN**: 添加一条独立语句或声明：`"a requires expression must contain at least one requirement">;`。
- **L917 EN**: Declares TableGen def record `err_requires_expr_parameter_list_ellipsis`.
  **L917 CN**: 声明 TableGen def 记录 `err_requires_expr_parameter_list_ellipsis`。
- **L918 EN**: Adds a standalone statement or declaration: `"varargs not allowed in requires expression">;`.
  **L918 CN**: 添加一条独立语句或声明：`"varargs not allowed in requires expression">;`。
- **L919 EN**: Declares TableGen def record `err_requires_expr_explicit_object_parameter`.
  **L919 CN**: 声明 TableGen def 记录 `err_requires_expr_explicit_object_parameter`。
- **L920 EN**: Adds a standalone statement or declaration: `"a requires expression cannot have an explicit object parameter">;`.
  **L920 CN**: 添加一条独立语句或声明：`"a requires expression cannot have an explicit object parameter">;`。
- **L921 EN**: Declares TableGen def record `err_expected_semi_requirement`.
  **L921 CN**: 声明 TableGen def 记录 `err_expected_semi_requirement`。
- **L922 EN**: Adds a standalone statement or declaration: `"expected ';' at end of requirement">;`.
  **L922 CN**: 添加一条独立语句或声明：`"expected ';' at end of requirement">;`。
- **L923 EN**: Declares TableGen def record `err_requires_expr_missing_arrow`.
  **L923 CN**: 声明 TableGen def 记录 `err_requires_expr_missing_arrow`。
- **L924 EN**: Adds a standalone statement or declaration: `"expected '->' before expression type requirement">;`.
  **L924 CN**: 添加一条独立语句或声明：`"expected '->' before expression type requirement">;`。
- **L925 EN**: Declares TableGen def record `err_requires_expr_expected_type_constraint`.
  **L925 CN**: 声明 TableGen def 记录 `err_requires_expr_expected_type_constraint`。
- **L926 EN**: Adds a standalone statement or declaration: `"expected concept name with optional arguments">;`.
  **L926 CN**: 添加一条独立语句或声明：`"expected concept name with optional arguments">;`。
- **L927 EN**: Declares TableGen def record `err_requires_expr_simple_requirement_noexcept`.
  **L927 CN**: 声明 TableGen def 记录 `err_requires_expr_simple_requirement_noexcept`。
- **L928 EN**: Continues logic associated with callable symbol `requirement`.
  **L928 CN**: 继续与可调用符号 `requirement` 相关的逻辑。
- **L929 EN**: Adds a standalone statement or declaration: `"the expression)">;`.
  **L929 CN**: 添加一条独立语句或声明：`"the expression)">;`。
- **L930 EN**: Declares TableGen def record `err_requires_expr_in_simple_requirement`.
  **L930 CN**: 声明 TableGen def 记录 `err_requires_expr_in_simple_requirement`。
- **L931 EN**: Continues the surrounding expression or declaration: `"requires expression in requirement body; did "`.
  **L931 CN**: 继续构造周围的表达式或声明：`"requires expression in requirement body; did "`。
- **L932 EN**: Continues the surrounding expression or declaration: `"you intend to place it in a nested requirement? (add another 'requires' "`.
  **L932 CN**: 继续构造周围的表达式或声明：`"you intend to place it in a nested requirement? (add another 'requires' "`。
- **L933 EN**: Adds a standalone statement or declaration: `"before the expression)">;`.
  **L933 CN**: 添加一条独立语句或声明：`"before the expression)">;`。
- **L934 EN**: Declares TableGen def record `missing_template_arg_list_after_template_kw`.
  **L934 CN**: 声明 TableGen def 记录 `missing_template_arg_list_after_template_kw`。
- **L935 EN**: Continues the surrounding expression or declaration: `"a template argument list is expected after a name prefixed by the template "`.
  **L935 CN**: 继续构造周围的表达式或声明：`"a template argument list is expected after a name prefixed by the template "`。
- **L936 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"keyword">, InGroup<DiagGroup<"missing-template-arg-list-after-template-kw">>,`.
  **L936 CN**: 继续一个多行参数列表、初始化器或聚合项：`"keyword">, InGroup<DiagGroup<"missing-template-arg-list-after-template-kw">>,`。

### Lines 937-960

````tablegen
  DefaultError;

def err_missing_dependent_template_keyword : Error<
  "use 'template' keyword to treat '%0' as a dependent template name">;
def warn_missing_dependent_template_keyword : ExtWarn<
  "use 'template' keyword to treat '%0' as a dependent template name">;

def err_cxx26_template_template_params
    : Error<"%select{variable template|concept}0 template parameter is a C++2c "
            "extension">;

def ext_extern_template : Extension<
  "extern templates are a C++11 extension">, InGroup<CXX11>;
def warn_cxx98_compat_extern_template : Warning<
  "extern templates are incompatible with C++98">,
  InGroup<CXX98CompatPedantic>, DefaultIgnore;
def warn_static_inline_explicit_inst_ignored : Warning<
  "ignoring '%select{static|inline}0' keyword on explicit template "
  "instantiation">, InGroup<DiagGroup<"static-inline-explicit-instantiation">>;

// Constructor template diagnostics.
def err_out_of_line_constructor_template_id : Error<
  "out-of-line constructor for %0 cannot have template arguments">;

````
- **L937 EN**: Adds a standalone statement or declaration: `DefaultError;`.
  **L937 CN**: 添加一条独立语句或声明：`DefaultError;`。
- **L938 EN**: Blank line separating nearby declarations or logic blocks.
  **L938 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L939 EN**: Declares TableGen def record `err_missing_dependent_template_keyword`.
  **L939 CN**: 声明 TableGen def 记录 `err_missing_dependent_template_keyword`。
- **L940 EN**: Adds a standalone statement or declaration: `"use 'template' keyword to treat '%0' as a dependent template name">;`.
  **L940 CN**: 添加一条独立语句或声明：`"use 'template' keyword to treat '%0' as a dependent template name">;`。
- **L941 EN**: Declares TableGen def record `warn_missing_dependent_template_keyword`.
  **L941 CN**: 声明 TableGen def 记录 `warn_missing_dependent_template_keyword`。
- **L942 EN**: Adds a standalone statement or declaration: `"use 'template' keyword to treat '%0' as a dependent template name">;`.
  **L942 CN**: 添加一条独立语句或声明：`"use 'template' keyword to treat '%0' as a dependent template name">;`。
- **L943 EN**: Blank line separating nearby declarations or logic blocks.
  **L943 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L944 EN**: Declares TableGen def record `err_cxx26_template_template_params`.
  **L944 CN**: 声明 TableGen def 记录 `err_cxx26_template_template_params`。
- **L945 EN**: Continues the surrounding expression or declaration: `: Error<"%select{variable template|concept}0 template parameter is a C++2c "`.
  **L945 CN**: 继续构造周围的表达式或声明：`: Error<"%select{variable template|concept}0 template parameter is a C++2c "`。
- **L946 EN**: Adds a standalone statement or declaration: `"extension">;`.
  **L946 CN**: 添加一条独立语句或声明：`"extension">;`。
- **L947 EN**: Blank line separating nearby declarations or logic blocks.
  **L947 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L948 EN**: Declares TableGen def record `ext_extern_template`.
  **L948 CN**: 声明 TableGen def 记录 `ext_extern_template`。
- **L949 EN**: Adds a standalone statement or declaration: `"extern templates are a C++11 extension">, InGroup<CXX11>;`.
  **L949 CN**: 添加一条独立语句或声明：`"extern templates are a C++11 extension">, InGroup<CXX11>;`。
- **L950 EN**: Declares TableGen def record `warn_cxx98_compat_extern_template`.
  **L950 CN**: 声明 TableGen def 记录 `warn_cxx98_compat_extern_template`。
- **L951 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"extern templates are incompatible with C++98">,`.
  **L951 CN**: 继续一个多行参数列表、初始化器或聚合项：`"extern templates are incompatible with C++98">,`。
- **L952 EN**: Adds a standalone statement or declaration: `InGroup<CXX98CompatPedantic>, DefaultIgnore;`.
  **L952 CN**: 添加一条独立语句或声明：`InGroup<CXX98CompatPedantic>, DefaultIgnore;`。
- **L953 EN**: Declares TableGen def record `warn_static_inline_explicit_inst_ignored`.
  **L953 CN**: 声明 TableGen def 记录 `warn_static_inline_explicit_inst_ignored`。
- **L954 EN**: Continues the surrounding expression or declaration: `"ignoring '%select{static|inline}0' keyword on explicit template "`.
  **L954 CN**: 继续构造周围的表达式或声明：`"ignoring '%select{static|inline}0' keyword on explicit template "`。
- **L955 EN**: Adds a standalone statement or declaration: `"instantiation">, InGroup<DiagGroup<"static-inline-explicit-instantiation">>;`.
  **L955 CN**: 添加一条独立语句或声明：`"instantiation">, InGroup<DiagGroup<"static-inline-explicit-instantiation">>;`。
- **L956 EN**: Blank line separating nearby declarations or logic blocks.
  **L956 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L957 EN**: Comment explains nearby logic, constraints, or intent: `Constructor template diagnostics.`.
  **L957 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Constructor template diagnostics.`。
- **L958 EN**: Declares TableGen def record `err_out_of_line_constructor_template_id`.
  **L958 CN**: 声明 TableGen def 记录 `err_out_of_line_constructor_template_id`。
- **L959 EN**: Adds a standalone statement or declaration: `"out-of-line constructor for %0 cannot have template arguments">;`.
  **L959 CN**: 添加一条独立语句或声明：`"out-of-line constructor for %0 cannot have template arguments">;`。
- **L960 EN**: Blank line separating nearby declarations or logic blocks.
  **L960 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 961-984

````tablegen
def err_expected_qualified_after_typename : Error<
  "expected a qualified name after 'typename'">;
def warn_expected_qualified_after_typename : ExtWarn<
  "expected a qualified name after 'typename'">;

def err_typename_refers_to_non_type_template : Error<
  "typename specifier refers to a non-type template">;
def err_expected_type_name_after_typename : Error<
  "expected an identifier or template-id after '::'">;
def err_explicit_spec_non_template : Error<
  "explicit %select{specialization|instantiation}0 of "
  "%select{non-|undeclared }3template %1 %2">;

def err_default_template_template_parameter_not_template : Error<
  "default template argument for a template template parameter must be a class "
  "template">;

def ext_fold_expression : ExtWarn<
  "pack fold expression is a C++17 extension">,
  InGroup<CXX17>;
def warn_cxx14_compat_fold_expression : Warning<
  "pack fold expression is incompatible with C++ standards before C++17">,
  InGroup<CXXPre17Compat>, DefaultIgnore;
def err_expected_fold_operator : Error<
````
- **L961 EN**: Declares TableGen def record `err_expected_qualified_after_typename`.
  **L961 CN**: 声明 TableGen def 记录 `err_expected_qualified_after_typename`。
- **L962 EN**: Adds a standalone statement or declaration: `"expected a qualified name after 'typename'">;`.
  **L962 CN**: 添加一条独立语句或声明：`"expected a qualified name after 'typename'">;`。
- **L963 EN**: Declares TableGen def record `warn_expected_qualified_after_typename`.
  **L963 CN**: 声明 TableGen def 记录 `warn_expected_qualified_after_typename`。
- **L964 EN**: Adds a standalone statement or declaration: `"expected a qualified name after 'typename'">;`.
  **L964 CN**: 添加一条独立语句或声明：`"expected a qualified name after 'typename'">;`。
- **L965 EN**: Blank line separating nearby declarations or logic blocks.
  **L965 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L966 EN**: Declares TableGen def record `err_typename_refers_to_non_type_template`.
  **L966 CN**: 声明 TableGen def 记录 `err_typename_refers_to_non_type_template`。
- **L967 EN**: Adds a standalone statement or declaration: `"typename specifier refers to a non-type template">;`.
  **L967 CN**: 添加一条独立语句或声明：`"typename specifier refers to a non-type template">;`。
- **L968 EN**: Declares TableGen def record `err_expected_type_name_after_typename`.
  **L968 CN**: 声明 TableGen def 记录 `err_expected_type_name_after_typename`。
- **L969 EN**: Adds a standalone statement or declaration: `"expected an identifier or template-id after '::'">;`.
  **L969 CN**: 添加一条独立语句或声明：`"expected an identifier or template-id after '::'">;`。
- **L970 EN**: Declares TableGen def record `err_explicit_spec_non_template`.
  **L970 CN**: 声明 TableGen def 记录 `err_explicit_spec_non_template`。
- **L971 EN**: Continues the surrounding expression or declaration: `"explicit %select{specialization|instantiation}0 of "`.
  **L971 CN**: 继续构造周围的表达式或声明：`"explicit %select{specialization|instantiation}0 of "`。
- **L972 EN**: Adds a standalone statement or declaration: `"%select{non-|undeclared }3template %1 %2">;`.
  **L972 CN**: 添加一条独立语句或声明：`"%select{non-|undeclared }3template %1 %2">;`。
- **L973 EN**: Blank line separating nearby declarations or logic blocks.
  **L973 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L974 EN**: Declares TableGen def record `err_default_template_template_parameter_not_template`.
  **L974 CN**: 声明 TableGen def 记录 `err_default_template_template_parameter_not_template`。
- **L975 EN**: Continues the surrounding expression or declaration: `"default template argument for a template template parameter must be a class "`.
  **L975 CN**: 继续构造周围的表达式或声明：`"default template argument for a template template parameter must be a class "`。
- **L976 EN**: Adds a standalone statement or declaration: `"template">;`.
  **L976 CN**: 添加一条独立语句或声明：`"template">;`。
- **L977 EN**: Blank line separating nearby declarations or logic blocks.
  **L977 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L978 EN**: Declares TableGen def record `ext_fold_expression`.
  **L978 CN**: 声明 TableGen def 记录 `ext_fold_expression`。
- **L979 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"pack fold expression is a C++17 extension">,`.
  **L979 CN**: 继续一个多行参数列表、初始化器或聚合项：`"pack fold expression is a C++17 extension">,`。
- **L980 EN**: Adds a standalone statement or declaration: `InGroup<CXX17>;`.
  **L980 CN**: 添加一条独立语句或声明：`InGroup<CXX17>;`。
- **L981 EN**: Declares TableGen def record `warn_cxx14_compat_fold_expression`.
  **L981 CN**: 声明 TableGen def 记录 `warn_cxx14_compat_fold_expression`。
- **L982 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"pack fold expression is incompatible with C++ standards before C++17">,`.
  **L982 CN**: 继续一个多行参数列表、初始化器或聚合项：`"pack fold expression is incompatible with C++ standards before C++17">,`。
- **L983 EN**: Adds a standalone statement or declaration: `InGroup<CXXPre17Compat>, DefaultIgnore;`.
  **L983 CN**: 添加一条独立语句或声明：`InGroup<CXXPre17Compat>, DefaultIgnore;`。
- **L984 EN**: Declares TableGen def record `err_expected_fold_operator`.
  **L984 CN**: 声明 TableGen def 记录 `err_expected_fold_operator`。

### Lines 985-1008

````tablegen
  "expected a foldable binary operator in fold expression">;
def err_fold_operator_mismatch : Error<
  "operators in fold expression must be the same">;

def err_ctor_init_missing_comma : Error<
  "missing ',' between base or member initializers">;

// C++ declarations
def err_friend_decl_defines_type : Error<
  "cannot define a type in a friend declaration">;
def err_missing_whitespace_digraph : Error<
  "found '<::' after a "
  "%select{template name|addrspace_cast|const_cast|dynamic_cast|reinterpret_cast|static_cast}0"
  " which forms the digraph '<:' (aka '[') and a ':', did you mean '< ::'?">;

def ext_defaulted_deleted_function : ExtWarn<
  "%select{defaulted|deleted}0 function definitions are a C++11 extension">,
  InGroup<CXX11>;
def warn_cxx98_compat_defaulted_deleted_function : Warning<
  "%select{defaulted|deleted}0 function definitions are incompatible with C++98">,
  InGroup<CXX98Compat>, DefaultIgnore;

def ext_delete_with_message : Extension<
  "'= delete' with a message is a C++2c extension">, InGroup<CXX26>;
````
- **L985 EN**: Adds a standalone statement or declaration: `"expected a foldable binary operator in fold expression">;`.
  **L985 CN**: 添加一条独立语句或声明：`"expected a foldable binary operator in fold expression">;`。
- **L986 EN**: Declares TableGen def record `err_fold_operator_mismatch`.
  **L986 CN**: 声明 TableGen def 记录 `err_fold_operator_mismatch`。
- **L987 EN**: Adds a standalone statement or declaration: `"operators in fold expression must be the same">;`.
  **L987 CN**: 添加一条独立语句或声明：`"operators in fold expression must be the same">;`。
- **L988 EN**: Blank line separating nearby declarations or logic blocks.
  **L988 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L989 EN**: Declares TableGen def record `err_ctor_init_missing_comma`.
  **L989 CN**: 声明 TableGen def 记录 `err_ctor_init_missing_comma`。
- **L990 EN**: Adds a standalone statement or declaration: `"missing ',' between base or member initializers">;`.
  **L990 CN**: 添加一条独立语句或声明：`"missing ',' between base or member initializers">;`。
- **L991 EN**: Blank line separating nearby declarations or logic blocks.
  **L991 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L992 EN**: Comment explains nearby logic, constraints, or intent: `C++ declarations`.
  **L992 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C++ declarations`。
- **L993 EN**: Declares TableGen def record `err_friend_decl_defines_type`.
  **L993 CN**: 声明 TableGen def 记录 `err_friend_decl_defines_type`。
- **L994 EN**: Adds a standalone statement or declaration: `"cannot define a type in a friend declaration">;`.
  **L994 CN**: 添加一条独立语句或声明：`"cannot define a type in a friend declaration">;`。
- **L995 EN**: Declares TableGen def record `err_missing_whitespace_digraph`.
  **L995 CN**: 声明 TableGen def 记录 `err_missing_whitespace_digraph`。
- **L996 EN**: Continues the surrounding expression or declaration: `"found '<::' after a "`.
  **L996 CN**: 继续构造周围的表达式或声明：`"found '<::' after a "`。
- **L997 EN**: Continues the surrounding expression or declaration: `"%select{template name|addrspace_cast|const_cast|dynamic_cast|reinterpret_cast|static_cast}0"`.
  **L997 CN**: 继续构造周围的表达式或声明：`"%select{template name|addrspace_cast|const_cast|dynamic_cast|reinterpret_cast|static_cast}0"`。
- **L998 EN**: Executes a call or declaration centered on `'<:'`.
  **L998 CN**: 执行以 `'<:'` 为核心的调用或声明。
- **L999 EN**: Blank line separating nearby declarations or logic blocks.
  **L999 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1000 EN**: Declares TableGen def record `ext_defaulted_deleted_function`.
  **L1000 CN**: 声明 TableGen def 记录 `ext_defaulted_deleted_function`。
- **L1001 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"%select{defaulted|deleted}0 function definitions are a C++11 extension">,`.
  **L1001 CN**: 继续一个多行参数列表、初始化器或聚合项：`"%select{defaulted|deleted}0 function definitions are a C++11 extension">,`。
- **L1002 EN**: Adds a standalone statement or declaration: `InGroup<CXX11>;`.
  **L1002 CN**: 添加一条独立语句或声明：`InGroup<CXX11>;`。
- **L1003 EN**: Declares TableGen def record `warn_cxx98_compat_defaulted_deleted_function`.
  **L1003 CN**: 声明 TableGen def 记录 `warn_cxx98_compat_defaulted_deleted_function`。
- **L1004 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"%select{defaulted|deleted}0 function definitions are incompatible with C++98">,`.
  **L1004 CN**: 继续一个多行参数列表、初始化器或聚合项：`"%select{defaulted|deleted}0 function definitions are incompatible with C++98">,`。
- **L1005 EN**: Adds a standalone statement or declaration: `InGroup<CXX98Compat>, DefaultIgnore;`.
  **L1005 CN**: 添加一条独立语句或声明：`InGroup<CXX98Compat>, DefaultIgnore;`。
- **L1006 EN**: Blank line separating nearby declarations or logic blocks.
  **L1006 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1007 EN**: Declares TableGen def record `ext_delete_with_message`.
  **L1007 CN**: 声明 TableGen def 记录 `ext_delete_with_message`。
- **L1008 EN**: Adds a standalone statement or declaration: `"'= delete' with a message is a C++2c extension">, InGroup<CXX26>;`.
  **L1008 CN**: 添加一条独立语句或声明：`"'= delete' with a message is a C++2c extension">, InGroup<CXX26>;`。

### Lines 1009-1032

````tablegen
def warn_cxx23_delete_with_message : Warning<
  "'= delete' with a message is incompatible with C++ standards before C++2c">,
  DefaultIgnore, InGroup<CXXPre26Compat>;

def ext_variadic_friends : ExtWarn<
  "variadic 'friend' declarations are a C++2c extension">, InGroup<CXX26>;
def warn_cxx23_variadic_friends : Warning<
  "variadic 'friend' declarations are incompatible with C++ standards before C++2c">,
  DefaultIgnore, InGroup<CXXPre26Compat>;

def err_friend_concept : Error<
  "friend declaration cannot be a concept">;

// C++11 default member initialization
def ext_nonstatic_member_init : ExtWarn<
  "default member initializer for non-static data member is a C++11 "
  "extension">, InGroup<CXX11>;
def warn_cxx98_compat_nonstatic_member_init : Warning<
  "default member initializer for non-static data members is incompatible with "
  "C++98">, InGroup<CXX98Compat>, DefaultIgnore;
def ext_bitfield_member_init: ExtWarn<
  "default member initializer for bit-field is a C++20 extension">,
  InGroup<CXX20>;
def warn_cxx17_compat_bitfield_member_init: Warning<
````
- **L1009 EN**: Declares TableGen def record `warn_cxx23_delete_with_message`.
  **L1009 CN**: 声明 TableGen def 记录 `warn_cxx23_delete_with_message`。
- **L1010 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'= delete' with a message is incompatible with C++ standards before C++2c">,`.
  **L1010 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'= delete' with a message is incompatible with C++ standards before C++2c">,`。
- **L1011 EN**: Adds a standalone statement or declaration: `DefaultIgnore, InGroup<CXXPre26Compat>;`.
  **L1011 CN**: 添加一条独立语句或声明：`DefaultIgnore, InGroup<CXXPre26Compat>;`。
- **L1012 EN**: Blank line separating nearby declarations or logic blocks.
  **L1012 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1013 EN**: Declares TableGen def record `ext_variadic_friends`.
  **L1013 CN**: 声明 TableGen def 记录 `ext_variadic_friends`。
- **L1014 EN**: Adds a standalone statement or declaration: `"variadic 'friend' declarations are a C++2c extension">, InGroup<CXX26>;`.
  **L1014 CN**: 添加一条独立语句或声明：`"variadic 'friend' declarations are a C++2c extension">, InGroup<CXX26>;`。
- **L1015 EN**: Declares TableGen def record `warn_cxx23_variadic_friends`.
  **L1015 CN**: 声明 TableGen def 记录 `warn_cxx23_variadic_friends`。
- **L1016 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"variadic 'friend' declarations are incompatible with C++ standards before C++2c">,`.
  **L1016 CN**: 继续一个多行参数列表、初始化器或聚合项：`"variadic 'friend' declarations are incompatible with C++ standards before C++2c">,`。
- **L1017 EN**: Adds a standalone statement or declaration: `DefaultIgnore, InGroup<CXXPre26Compat>;`.
  **L1017 CN**: 添加一条独立语句或声明：`DefaultIgnore, InGroup<CXXPre26Compat>;`。
- **L1018 EN**: Blank line separating nearby declarations or logic blocks.
  **L1018 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1019 EN**: Declares TableGen def record `err_friend_concept`.
  **L1019 CN**: 声明 TableGen def 记录 `err_friend_concept`。
- **L1020 EN**: Adds a standalone statement or declaration: `"friend declaration cannot be a concept">;`.
  **L1020 CN**: 添加一条独立语句或声明：`"friend declaration cannot be a concept">;`。
- **L1021 EN**: Blank line separating nearby declarations or logic blocks.
  **L1021 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1022 EN**: Comment explains nearby logic, constraints, or intent: `C++11 default member initialization`.
  **L1022 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C++11 default member initialization`。
- **L1023 EN**: Declares TableGen def record `ext_nonstatic_member_init`.
  **L1023 CN**: 声明 TableGen def 记录 `ext_nonstatic_member_init`。
- **L1024 EN**: Continues the surrounding expression or declaration: `"default member initializer for non-static data member is a C++11 "`.
  **L1024 CN**: 继续构造周围的表达式或声明：`"default member initializer for non-static data member is a C++11 "`。
- **L1025 EN**: Adds a standalone statement or declaration: `"extension">, InGroup<CXX11>;`.
  **L1025 CN**: 添加一条独立语句或声明：`"extension">, InGroup<CXX11>;`。
- **L1026 EN**: Declares TableGen def record `warn_cxx98_compat_nonstatic_member_init`.
  **L1026 CN**: 声明 TableGen def 记录 `warn_cxx98_compat_nonstatic_member_init`。
- **L1027 EN**: Continues the surrounding expression or declaration: `"default member initializer for non-static data members is incompatible with "`.
  **L1027 CN**: 继续构造周围的表达式或声明：`"default member initializer for non-static data members is incompatible with "`。
- **L1028 EN**: Adds a standalone statement or declaration: `"C++98">, InGroup<CXX98Compat>, DefaultIgnore;`.
  **L1028 CN**: 添加一条独立语句或声明：`"C++98">, InGroup<CXX98Compat>, DefaultIgnore;`。
- **L1029 EN**: Declares TableGen def record `ext_bitfield_member_init`.
  **L1029 CN**: 声明 TableGen def 记录 `ext_bitfield_member_init`。
- **L1030 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"default member initializer for bit-field is a C++20 extension">,`.
  **L1030 CN**: 继续一个多行参数列表、初始化器或聚合项：`"default member initializer for bit-field is a C++20 extension">,`。
- **L1031 EN**: Adds a standalone statement or declaration: `InGroup<CXX20>;`.
  **L1031 CN**: 添加一条独立语句或声明：`InGroup<CXX20>;`。
- **L1032 EN**: Declares TableGen def record `warn_cxx17_compat_bitfield_member_init`.
  **L1032 CN**: 声明 TableGen def 记录 `warn_cxx17_compat_bitfield_member_init`。

### Lines 1033-1056

````tablegen
  "default member initializer for bit-field is incompatible with "
  "C++ standards before C++20">, InGroup<CXXPre20Compat>, DefaultIgnore;
def err_anon_bitfield_member_init : Error<
  "anonymous bit-field cannot have a default member initializer">;
def err_incomplete_array_member_init: Error<
  "array bound cannot be deduced from a default member initializer">;

// C++11 alias-declaration
def ext_alias_declaration : ExtWarn<
  "alias declarations are a C++11 extension">, InGroup<CXX11>;
def warn_cxx98_compat_alias_declaration : Warning<
  "alias declarations are incompatible with C++98">,
  InGroup<CXX98Compat>, DefaultIgnore;
def err_alias_declaration_not_identifier : Error<
  "name defined in alias declaration must be an identifier">;
def err_alias_declaration_specialization : Error<
  "%select{partial specialization|explicit specialization|explicit instantiation}0 of alias templates is not permitted">;
def err_alias_declaration_pack_expansion : Error<
  "alias declaration cannot be a pack expansion">;

// C++17 using-declaration pack expansions
def ext_multi_using_declaration : ExtWarn<
  "use of multiple declarators in a single using declaration is "
  "a C++17 extension">, InGroup<CXX17>;
````
- **L1033 EN**: Continues the surrounding expression or declaration: `"default member initializer for bit-field is incompatible with "`.
  **L1033 CN**: 继续构造周围的表达式或声明：`"default member initializer for bit-field is incompatible with "`。
- **L1034 EN**: Adds a standalone statement or declaration: `"C++ standards before C++20">, InGroup<CXXPre20Compat>, DefaultIgnore;`.
  **L1034 CN**: 添加一条独立语句或声明：`"C++ standards before C++20">, InGroup<CXXPre20Compat>, DefaultIgnore;`。
- **L1035 EN**: Declares TableGen def record `err_anon_bitfield_member_init`.
  **L1035 CN**: 声明 TableGen def 记录 `err_anon_bitfield_member_init`。
- **L1036 EN**: Adds a standalone statement or declaration: `"anonymous bit-field cannot have a default member initializer">;`.
  **L1036 CN**: 添加一条独立语句或声明：`"anonymous bit-field cannot have a default member initializer">;`。
- **L1037 EN**: Declares TableGen def record `err_incomplete_array_member_init`.
  **L1037 CN**: 声明 TableGen def 记录 `err_incomplete_array_member_init`。
- **L1038 EN**: Adds a standalone statement or declaration: `"array bound cannot be deduced from a default member initializer">;`.
  **L1038 CN**: 添加一条独立语句或声明：`"array bound cannot be deduced from a default member initializer">;`。
- **L1039 EN**: Blank line separating nearby declarations or logic blocks.
  **L1039 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1040 EN**: Comment explains nearby logic, constraints, or intent: `C++11 alias-declaration`.
  **L1040 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C++11 alias-declaration`。
- **L1041 EN**: Declares TableGen def record `ext_alias_declaration`.
  **L1041 CN**: 声明 TableGen def 记录 `ext_alias_declaration`。
- **L1042 EN**: Adds a standalone statement or declaration: `"alias declarations are a C++11 extension">, InGroup<CXX11>;`.
  **L1042 CN**: 添加一条独立语句或声明：`"alias declarations are a C++11 extension">, InGroup<CXX11>;`。
- **L1043 EN**: Declares TableGen def record `warn_cxx98_compat_alias_declaration`.
  **L1043 CN**: 声明 TableGen def 记录 `warn_cxx98_compat_alias_declaration`。
- **L1044 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"alias declarations are incompatible with C++98">,`.
  **L1044 CN**: 继续一个多行参数列表、初始化器或聚合项：`"alias declarations are incompatible with C++98">,`。
- **L1045 EN**: Adds a standalone statement or declaration: `InGroup<CXX98Compat>, DefaultIgnore;`.
  **L1045 CN**: 添加一条独立语句或声明：`InGroup<CXX98Compat>, DefaultIgnore;`。
- **L1046 EN**: Declares TableGen def record `err_alias_declaration_not_identifier`.
  **L1046 CN**: 声明 TableGen def 记录 `err_alias_declaration_not_identifier`。
- **L1047 EN**: Adds a standalone statement or declaration: `"name defined in alias declaration must be an identifier">;`.
  **L1047 CN**: 添加一条独立语句或声明：`"name defined in alias declaration must be an identifier">;`。
- **L1048 EN**: Declares TableGen def record `err_alias_declaration_specialization`.
  **L1048 CN**: 声明 TableGen def 记录 `err_alias_declaration_specialization`。
- **L1049 EN**: Adds a standalone statement or declaration: `"%select{partial specialization|explicit specialization|explicit instantiation}0 of alias templates is not permitted">;`.
  **L1049 CN**: 添加一条独立语句或声明：`"%select{partial specialization|explicit specialization|explicit instantiation}0 of alias templates is not permitted">;`。
- **L1050 EN**: Declares TableGen def record `err_alias_declaration_pack_expansion`.
  **L1050 CN**: 声明 TableGen def 记录 `err_alias_declaration_pack_expansion`。
- **L1051 EN**: Adds a standalone statement or declaration: `"alias declaration cannot be a pack expansion">;`.
  **L1051 CN**: 添加一条独立语句或声明：`"alias declaration cannot be a pack expansion">;`。
- **L1052 EN**: Blank line separating nearby declarations or logic blocks.
  **L1052 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1053 EN**: Comment explains nearby logic, constraints, or intent: `C++17 using-declaration pack expansions`.
  **L1053 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C++17 using-declaration pack expansions`。
- **L1054 EN**: Declares TableGen def record `ext_multi_using_declaration`.
  **L1054 CN**: 声明 TableGen def 记录 `ext_multi_using_declaration`。
- **L1055 EN**: Continues the surrounding expression or declaration: `"use of multiple declarators in a single using declaration is "`.
  **L1055 CN**: 继续构造周围的表达式或声明：`"use of multiple declarators in a single using declaration is "`。
- **L1056 EN**: Adds a standalone statement or declaration: `"a C++17 extension">, InGroup<CXX17>;`.
  **L1056 CN**: 添加一条独立语句或声明：`"a C++17 extension">, InGroup<CXX17>;`。

### Lines 1057-1080

````tablegen
def warn_cxx17_compat_multi_using_declaration : Warning<
  "use of multiple declarators in a single using declaration is "
  "incompatible with C++ standards before C++17">,
  InGroup<CXXPre17Compat>, DefaultIgnore;
def ext_using_declaration_pack : ExtWarn<
  "pack expansion of using declaration is a C++17 extension">, InGroup<CXX17>;
def warn_cxx17_compat_using_declaration_pack : Warning<
  "pack expansion using declaration is incompatible with C++ standards "
  "before C++17">, InGroup<CXXPre17Compat>, DefaultIgnore;

// C++11 override control
def ext_override_control_keyword : ExtWarn<
  "'%0' keyword is a C++11 extension">, InGroup<CXX11>;
def warn_cxx98_compat_override_control_keyword : Warning<
  "'%0' keyword is incompatible with C++98">,
  InGroup<CXX98Compat>, DefaultIgnore;
def err_override_control_interface : Error<
  "'%0' keyword not permitted with interface types">;
def ext_ms_sealed_keyword : ExtWarn<
  "'sealed' keyword is a Microsoft extension">,
  InGroup<MicrosoftSealed>;
def ext_ms_abstract_keyword : ExtWarn<
  "'abstract' keyword is a Microsoft extension">,
  InGroup<MicrosoftAbstract>;
````
- **L1057 EN**: Declares TableGen def record `warn_cxx17_compat_multi_using_declaration`.
  **L1057 CN**: 声明 TableGen def 记录 `warn_cxx17_compat_multi_using_declaration`。
- **L1058 EN**: Continues the surrounding expression or declaration: `"use of multiple declarators in a single using declaration is "`.
  **L1058 CN**: 继续构造周围的表达式或声明：`"use of multiple declarators in a single using declaration is "`。
- **L1059 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"incompatible with C++ standards before C++17">,`.
  **L1059 CN**: 继续一个多行参数列表、初始化器或聚合项：`"incompatible with C++ standards before C++17">,`。
- **L1060 EN**: Adds a standalone statement or declaration: `InGroup<CXXPre17Compat>, DefaultIgnore;`.
  **L1060 CN**: 添加一条独立语句或声明：`InGroup<CXXPre17Compat>, DefaultIgnore;`。
- **L1061 EN**: Declares TableGen def record `ext_using_declaration_pack`.
  **L1061 CN**: 声明 TableGen def 记录 `ext_using_declaration_pack`。
- **L1062 EN**: Adds a standalone statement or declaration: `"pack expansion of using declaration is a C++17 extension">, InGroup<CXX17>;`.
  **L1062 CN**: 添加一条独立语句或声明：`"pack expansion of using declaration is a C++17 extension">, InGroup<CXX17>;`。
- **L1063 EN**: Declares TableGen def record `warn_cxx17_compat_using_declaration_pack`.
  **L1063 CN**: 声明 TableGen def 记录 `warn_cxx17_compat_using_declaration_pack`。
- **L1064 EN**: Continues the surrounding expression or declaration: `"pack expansion using declaration is incompatible with C++ standards "`.
  **L1064 CN**: 继续构造周围的表达式或声明：`"pack expansion using declaration is incompatible with C++ standards "`。
- **L1065 EN**: Adds a standalone statement or declaration: `"before C++17">, InGroup<CXXPre17Compat>, DefaultIgnore;`.
  **L1065 CN**: 添加一条独立语句或声明：`"before C++17">, InGroup<CXXPre17Compat>, DefaultIgnore;`。
- **L1066 EN**: Blank line separating nearby declarations or logic blocks.
  **L1066 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1067 EN**: Comment explains nearby logic, constraints, or intent: `C++11 override control`.
  **L1067 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C++11 override control`。
- **L1068 EN**: Declares TableGen def record `ext_override_control_keyword`.
  **L1068 CN**: 声明 TableGen def 记录 `ext_override_control_keyword`。
- **L1069 EN**: Adds a standalone statement or declaration: `"'%0' keyword is a C++11 extension">, InGroup<CXX11>;`.
  **L1069 CN**: 添加一条独立语句或声明：`"'%0' keyword is a C++11 extension">, InGroup<CXX11>;`。
- **L1070 EN**: Declares TableGen def record `warn_cxx98_compat_override_control_keyword`.
  **L1070 CN**: 声明 TableGen def 记录 `warn_cxx98_compat_override_control_keyword`。
- **L1071 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'%0' keyword is incompatible with C++98">,`.
  **L1071 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'%0' keyword is incompatible with C++98">,`。
- **L1072 EN**: Adds a standalone statement or declaration: `InGroup<CXX98Compat>, DefaultIgnore;`.
  **L1072 CN**: 添加一条独立语句或声明：`InGroup<CXX98Compat>, DefaultIgnore;`。
- **L1073 EN**: Declares TableGen def record `err_override_control_interface`.
  **L1073 CN**: 声明 TableGen def 记录 `err_override_control_interface`。
- **L1074 EN**: Adds a standalone statement or declaration: `"'%0' keyword not permitted with interface types">;`.
  **L1074 CN**: 添加一条独立语句或声明：`"'%0' keyword not permitted with interface types">;`。
- **L1075 EN**: Declares TableGen def record `ext_ms_sealed_keyword`.
  **L1075 CN**: 声明 TableGen def 记录 `ext_ms_sealed_keyword`。
- **L1076 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'sealed' keyword is a Microsoft extension">,`.
  **L1076 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'sealed' keyword is a Microsoft extension">,`。
- **L1077 EN**: Adds a standalone statement or declaration: `InGroup<MicrosoftSealed>;`.
  **L1077 CN**: 添加一条独立语句或声明：`InGroup<MicrosoftSealed>;`。
- **L1078 EN**: Declares TableGen def record `ext_ms_abstract_keyword`.
  **L1078 CN**: 声明 TableGen def 记录 `ext_ms_abstract_keyword`。
- **L1079 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'abstract' keyword is a Microsoft extension">,`.
  **L1079 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'abstract' keyword is a Microsoft extension">,`。
- **L1080 EN**: Adds a standalone statement or declaration: `InGroup<MicrosoftAbstract>;`.
  **L1080 CN**: 添加一条独立语句或声明：`InGroup<MicrosoftAbstract>;`。

### Lines 1081-1104

````tablegen

def err_access_specifier_interface : Error<
  "interface types cannot specify '%select{private|protected}0' access">;

def err_duplicate_class_virt_specifier : Error<
  "class already marked '%0'">;

def err_duplicate_virt_specifier : Error<
  "class member already marked '%0'">;

def err_virt_specifier_outside_class : Error<
  "'%0' specifier is not allowed outside a class definition">;

def err_expected_parameter_pack : Error<
  "expected the name of a parameter pack">;
def err_paren_sizeof_parameter_pack : Error<
  "missing parentheses around the size of parameter pack %0">;
def err_sizeof_parameter_pack : Error<
  "expected parenthesized parameter pack name in 'sizeof...' expression">;

// C++11 lambda expressions
def err_expected_comma_or_rsquare : Error<
  "expected ',' or ']' in lambda capture list">;
def err_this_captured_by_reference : Error<
````
- **L1081 EN**: Blank line separating nearby declarations or logic blocks.
  **L1081 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1082 EN**: Declares TableGen def record `err_access_specifier_interface`.
  **L1082 CN**: 声明 TableGen def 记录 `err_access_specifier_interface`。
- **L1083 EN**: Adds a standalone statement or declaration: `"interface types cannot specify '%select{private|protected}0' access">;`.
  **L1083 CN**: 添加一条独立语句或声明：`"interface types cannot specify '%select{private|protected}0' access">;`。
- **L1084 EN**: Blank line separating nearby declarations or logic blocks.
  **L1084 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1085 EN**: Declares TableGen def record `err_duplicate_class_virt_specifier`.
  **L1085 CN**: 声明 TableGen def 记录 `err_duplicate_class_virt_specifier`。
- **L1086 EN**: Adds a standalone statement or declaration: `"class already marked '%0'">;`.
  **L1086 CN**: 添加一条独立语句或声明：`"class already marked '%0'">;`。
- **L1087 EN**: Blank line separating nearby declarations or logic blocks.
  **L1087 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1088 EN**: Declares TableGen def record `err_duplicate_virt_specifier`.
  **L1088 CN**: 声明 TableGen def 记录 `err_duplicate_virt_specifier`。
- **L1089 EN**: Adds a standalone statement or declaration: `"class member already marked '%0'">;`.
  **L1089 CN**: 添加一条独立语句或声明：`"class member already marked '%0'">;`。
- **L1090 EN**: Blank line separating nearby declarations or logic blocks.
  **L1090 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1091 EN**: Declares TableGen def record `err_virt_specifier_outside_class`.
  **L1091 CN**: 声明 TableGen def 记录 `err_virt_specifier_outside_class`。
- **L1092 EN**: Adds a standalone statement or declaration: `"'%0' specifier is not allowed outside a class definition">;`.
  **L1092 CN**: 添加一条独立语句或声明：`"'%0' specifier is not allowed outside a class definition">;`。
- **L1093 EN**: Blank line separating nearby declarations or logic blocks.
  **L1093 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1094 EN**: Declares TableGen def record `err_expected_parameter_pack`.
  **L1094 CN**: 声明 TableGen def 记录 `err_expected_parameter_pack`。
- **L1095 EN**: Adds a standalone statement or declaration: `"expected the name of a parameter pack">;`.
  **L1095 CN**: 添加一条独立语句或声明：`"expected the name of a parameter pack">;`。
- **L1096 EN**: Declares TableGen def record `err_paren_sizeof_parameter_pack`.
  **L1096 CN**: 声明 TableGen def 记录 `err_paren_sizeof_parameter_pack`。
- **L1097 EN**: Adds a standalone statement or declaration: `"missing parentheses around the size of parameter pack %0">;`.
  **L1097 CN**: 添加一条独立语句或声明：`"missing parentheses around the size of parameter pack %0">;`。
- **L1098 EN**: Declares TableGen def record `err_sizeof_parameter_pack`.
  **L1098 CN**: 声明 TableGen def 记录 `err_sizeof_parameter_pack`。
- **L1099 EN**: Adds a standalone statement or declaration: `"expected parenthesized parameter pack name in 'sizeof...' expression">;`.
  **L1099 CN**: 添加一条独立语句或声明：`"expected parenthesized parameter pack name in 'sizeof...' expression">;`。
- **L1100 EN**: Blank line separating nearby declarations or logic blocks.
  **L1100 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1101 EN**: Comment explains nearby logic, constraints, or intent: `C++11 lambda expressions`.
  **L1101 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C++11 lambda expressions`。
- **L1102 EN**: Declares TableGen def record `err_expected_comma_or_rsquare`.
  **L1102 CN**: 声明 TableGen def 记录 `err_expected_comma_or_rsquare`。
- **L1103 EN**: Adds a standalone statement or declaration: `"expected ',' or ']' in lambda capture list">;`.
  **L1103 CN**: 添加一条独立语句或声明：`"expected ',' or ']' in lambda capture list">;`。
- **L1104 EN**: Declares TableGen def record `err_this_captured_by_reference`.
  **L1104 CN**: 声明 TableGen def 记录 `err_this_captured_by_reference`。

### Lines 1105-1128

````tablegen
  "'this' cannot be captured by reference">;
def err_expected_capture : Error<
  "expected variable name or 'this' in lambda capture list">;
def err_expected_lambda_body : Error<"expected body of lambda expression">;
def warn_cxx98_compat_lambda : Warning<
  "lambda expressions are incompatible with C++98">,
  InGroup<CXX98Compat>, DefaultIgnore;
def ext_lambda : ExtWarn<"lambdas are a %select{C++11|clang HLSL}0 extension">,
  InGroup<CXX11>;
def ext_hlsl_lambda : ExtWarn<ext_lambda.Summary>,
  InGroup<HLSLExtension>;
def err_lambda_decl_specifier_repeated : Error<
  "%select{'mutable'|'static'|'constexpr'|'consteval'}0 cannot "
  "appear multiple times in a lambda declarator">;
def err_lambda_capture_misplaced_ellipsis : Error<
  "ellipsis in pack %select{|init-}0capture must appear %select{after|before}0 "
  "the name of the capture">;
def err_lambda_capture_multiple_ellipses : Error<
  "multiple ellipses in pack capture">;
def err_binding_multiple_ellipses : Error<
  "multiple packs in structured binding declaration">;
def note_previous_ellipsis : Note<
  "previous binding pack specified here">;
def ext_cxx_binding_pack : ExtWarn<
````
- **L1105 EN**: Adds a standalone statement or declaration: `"'this' cannot be captured by reference">;`.
  **L1105 CN**: 添加一条独立语句或声明：`"'this' cannot be captured by reference">;`。
- **L1106 EN**: Declares TableGen def record `err_expected_capture`.
  **L1106 CN**: 声明 TableGen def 记录 `err_expected_capture`。
- **L1107 EN**: Adds a standalone statement or declaration: `"expected variable name or 'this' in lambda capture list">;`.
  **L1107 CN**: 添加一条独立语句或声明：`"expected variable name or 'this' in lambda capture list">;`。
- **L1108 EN**: Declares TableGen def record `err_expected_lambda_body`.
  **L1108 CN**: 声明 TableGen def 记录 `err_expected_lambda_body`。
- **L1109 EN**: Declares TableGen def record `warn_cxx98_compat_lambda`.
  **L1109 CN**: 声明 TableGen def 记录 `warn_cxx98_compat_lambda`。
- **L1110 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"lambda expressions are incompatible with C++98">,`.
  **L1110 CN**: 继续一个多行参数列表、初始化器或聚合项：`"lambda expressions are incompatible with C++98">,`。
- **L1111 EN**: Adds a standalone statement or declaration: `InGroup<CXX98Compat>, DefaultIgnore;`.
  **L1111 CN**: 添加一条独立语句或声明：`InGroup<CXX98Compat>, DefaultIgnore;`。
- **L1112 EN**: Declares TableGen def record `ext_lambda`.
  **L1112 CN**: 声明 TableGen def 记录 `ext_lambda`。
- **L1113 EN**: Adds a standalone statement or declaration: `InGroup<CXX11>;`.
  **L1113 CN**: 添加一条独立语句或声明：`InGroup<CXX11>;`。
- **L1114 EN**: Declares TableGen def record `ext_hlsl_lambda`.
  **L1114 CN**: 声明 TableGen def 记录 `ext_hlsl_lambda`。
- **L1115 EN**: Adds a standalone statement or declaration: `InGroup<HLSLExtension>;`.
  **L1115 CN**: 添加一条独立语句或声明：`InGroup<HLSLExtension>;`。
- **L1116 EN**: Declares TableGen def record `err_lambda_decl_specifier_repeated`.
  **L1116 CN**: 声明 TableGen def 记录 `err_lambda_decl_specifier_repeated`。
- **L1117 EN**: Continues the surrounding expression or declaration: `"%select{'mutable'|'static'|'constexpr'|'consteval'}0 cannot "`.
  **L1117 CN**: 继续构造周围的表达式或声明：`"%select{'mutable'|'static'|'constexpr'|'consteval'}0 cannot "`。
- **L1118 EN**: Adds a standalone statement or declaration: `"appear multiple times in a lambda declarator">;`.
  **L1118 CN**: 添加一条独立语句或声明：`"appear multiple times in a lambda declarator">;`。
- **L1119 EN**: Declares TableGen def record `err_lambda_capture_misplaced_ellipsis`.
  **L1119 CN**: 声明 TableGen def 记录 `err_lambda_capture_misplaced_ellipsis`。
- **L1120 EN**: Continues the surrounding expression or declaration: `"ellipsis in pack %select{|init-}0capture must appear %select{after|before}0 "`.
  **L1120 CN**: 继续构造周围的表达式或声明：`"ellipsis in pack %select{|init-}0capture must appear %select{after|before}0 "`。
- **L1121 EN**: Adds a standalone statement or declaration: `"the name of the capture">;`.
  **L1121 CN**: 添加一条独立语句或声明：`"the name of the capture">;`。
- **L1122 EN**: Declares TableGen def record `err_lambda_capture_multiple_ellipses`.
  **L1122 CN**: 声明 TableGen def 记录 `err_lambda_capture_multiple_ellipses`。
- **L1123 EN**: Adds a standalone statement or declaration: `"multiple ellipses in pack capture">;`.
  **L1123 CN**: 添加一条独立语句或声明：`"multiple ellipses in pack capture">;`。
- **L1124 EN**: Declares TableGen def record `err_binding_multiple_ellipses`.
  **L1124 CN**: 声明 TableGen def 记录 `err_binding_multiple_ellipses`。
- **L1125 EN**: Adds a standalone statement or declaration: `"multiple packs in structured binding declaration">;`.
  **L1125 CN**: 添加一条独立语句或声明：`"multiple packs in structured binding declaration">;`。
- **L1126 EN**: Declares TableGen def record `note_previous_ellipsis`.
  **L1126 CN**: 声明 TableGen def 记录 `note_previous_ellipsis`。
- **L1127 EN**: Adds a standalone statement or declaration: `"previous binding pack specified here">;`.
  **L1127 CN**: 添加一条独立语句或声明：`"previous binding pack specified here">;`。
- **L1128 EN**: Declares TableGen def record `ext_cxx_binding_pack`.
  **L1128 CN**: 声明 TableGen def 记录 `ext_cxx_binding_pack`。

### Lines 1129-1152

````tablegen
  "structured binding packs are a C++2c extension ">,
  InGroup<CXX26>;
def warn_cxx23_compat_binding_pack : Warning<
  "structured binding packs are incompatible with C++ standards before C++2c">,
  InGroup<CXXPre26Compat>, DefaultIgnore;
def err_capture_default_first : Error<
  "capture default must be first">;
def ext_decl_attrs_on_lambda : ExtWarn<
  "%select{an attribute specifier sequence|%1}0 in this position "
  "is a C++23 extension">, InGroup<CXX23AttrsOnLambda>;
def ext_lambda_missing_parens : ExtWarn<
  "lambda without a parameter clause is a C++23 extension">,
  InGroup<CXX23>;
def warn_cxx20_compat_decl_attrs_on_lambda : Warning<
  "%select{an attribute specifier sequence|%1}0 in this position "
  "is incompatible with C++ standards before C++23">,
  InGroup<CXXPre23Compat>, DefaultIgnore;

// C++17 lambda expressions
def err_expected_star_this_capture : Error<
  "expected 'this' following '*' in lambda capture list">;

// C++17 constexpr lambda expressions
def warn_cxx14_compat_constexpr_on_lambda : Warning<
````
- **L1129 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"structured binding packs are a C++2c extension ">,`.
  **L1129 CN**: 继续一个多行参数列表、初始化器或聚合项：`"structured binding packs are a C++2c extension ">,`。
- **L1130 EN**: Adds a standalone statement or declaration: `InGroup<CXX26>;`.
  **L1130 CN**: 添加一条独立语句或声明：`InGroup<CXX26>;`。
- **L1131 EN**: Declares TableGen def record `warn_cxx23_compat_binding_pack`.
  **L1131 CN**: 声明 TableGen def 记录 `warn_cxx23_compat_binding_pack`。
- **L1132 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"structured binding packs are incompatible with C++ standards before C++2c">,`.
  **L1132 CN**: 继续一个多行参数列表、初始化器或聚合项：`"structured binding packs are incompatible with C++ standards before C++2c">,`。
- **L1133 EN**: Adds a standalone statement or declaration: `InGroup<CXXPre26Compat>, DefaultIgnore;`.
  **L1133 CN**: 添加一条独立语句或声明：`InGroup<CXXPre26Compat>, DefaultIgnore;`。
- **L1134 EN**: Declares TableGen def record `err_capture_default_first`.
  **L1134 CN**: 声明 TableGen def 记录 `err_capture_default_first`。
- **L1135 EN**: Adds a standalone statement or declaration: `"capture default must be first">;`.
  **L1135 CN**: 添加一条独立语句或声明：`"capture default must be first">;`。
- **L1136 EN**: Declares TableGen def record `ext_decl_attrs_on_lambda`.
  **L1136 CN**: 声明 TableGen def 记录 `ext_decl_attrs_on_lambda`。
- **L1137 EN**: Continues the surrounding expression or declaration: `"%select{an attribute specifier sequence|%1}0 in this position "`.
  **L1137 CN**: 继续构造周围的表达式或声明：`"%select{an attribute specifier sequence|%1}0 in this position "`。
- **L1138 EN**: Adds a standalone statement or declaration: `"is a C++23 extension">, InGroup<CXX23AttrsOnLambda>;`.
  **L1138 CN**: 添加一条独立语句或声明：`"is a C++23 extension">, InGroup<CXX23AttrsOnLambda>;`。
- **L1139 EN**: Declares TableGen def record `ext_lambda_missing_parens`.
  **L1139 CN**: 声明 TableGen def 记录 `ext_lambda_missing_parens`。
- **L1140 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"lambda without a parameter clause is a C++23 extension">,`.
  **L1140 CN**: 继续一个多行参数列表、初始化器或聚合项：`"lambda without a parameter clause is a C++23 extension">,`。
- **L1141 EN**: Adds a standalone statement or declaration: `InGroup<CXX23>;`.
  **L1141 CN**: 添加一条独立语句或声明：`InGroup<CXX23>;`。
- **L1142 EN**: Declares TableGen def record `warn_cxx20_compat_decl_attrs_on_lambda`.
  **L1142 CN**: 声明 TableGen def 记录 `warn_cxx20_compat_decl_attrs_on_lambda`。
- **L1143 EN**: Continues the surrounding expression or declaration: `"%select{an attribute specifier sequence|%1}0 in this position "`.
  **L1143 CN**: 继续构造周围的表达式或声明：`"%select{an attribute specifier sequence|%1}0 in this position "`。
- **L1144 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"is incompatible with C++ standards before C++23">,`.
  **L1144 CN**: 继续一个多行参数列表、初始化器或聚合项：`"is incompatible with C++ standards before C++23">,`。
- **L1145 EN**: Adds a standalone statement or declaration: `InGroup<CXXPre23Compat>, DefaultIgnore;`.
  **L1145 CN**: 添加一条独立语句或声明：`InGroup<CXXPre23Compat>, DefaultIgnore;`。
- **L1146 EN**: Blank line separating nearby declarations or logic blocks.
  **L1146 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1147 EN**: Comment explains nearby logic, constraints, or intent: `C++17 lambda expressions`.
  **L1147 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C++17 lambda expressions`。
- **L1148 EN**: Declares TableGen def record `err_expected_star_this_capture`.
  **L1148 CN**: 声明 TableGen def 记录 `err_expected_star_this_capture`。
- **L1149 EN**: Adds a standalone statement or declaration: `"expected 'this' following '*' in lambda capture list">;`.
  **L1149 CN**: 添加一条独立语句或声明：`"expected 'this' following '*' in lambda capture list">;`。
- **L1150 EN**: Blank line separating nearby declarations or logic blocks.
  **L1150 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1151 EN**: Comment explains nearby logic, constraints, or intent: `C++17 constexpr lambda expressions`.
  **L1151 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C++17 constexpr lambda expressions`。
- **L1152 EN**: Declares TableGen def record `warn_cxx14_compat_constexpr_on_lambda`.
  **L1152 CN**: 声明 TableGen def 记录 `warn_cxx14_compat_constexpr_on_lambda`。

### Lines 1153-1176

````tablegen
  "constexpr on lambda expressions is incompatible with C++ standards before C++17">,
  InGroup<CXXPre17Compat>, DefaultIgnore;
def ext_constexpr_on_lambda_cxx17 : ExtWarn<
  "'constexpr' on lambda expressions is a C++17 extension">, InGroup<CXX17>;

// C++20 template lambdas
def ext_lambda_template_parameter_list: ExtWarn<
  "explicit template parameter list for lambdas is a C++20 extension">,
  InGroup<CXX20>;
def warn_cxx17_compat_lambda_template_parameter_list: Warning<
  "explicit template parameter list for lambdas is incompatible with "
  "C++ standards before C++20">, InGroup<CXXPre20Compat>, DefaultIgnore;
def err_lambda_template_parameter_list_empty : Error<
  "lambda template parameter list cannot be empty">;

// C++23 static lambdas
def err_static_lambda: ExtWarn<
  "static lambdas are a C++23 extension">, InGroup<CXX23>;
def warn_cxx20_compat_static_lambda : Warning<
  "static lambdas are incompatible with C++ standards before C++23">,
  InGroup<CXXPre23Compat>, DefaultIgnore;
def err_static_mutable_lambda : Error<
  "lambda cannot be both mutable and static">;
def err_static_lambda_captures : Error<
````
- **L1153 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"constexpr on lambda expressions is incompatible with C++ standards before C++17">,`.
  **L1153 CN**: 继续一个多行参数列表、初始化器或聚合项：`"constexpr on lambda expressions is incompatible with C++ standards before C++17">,`。
- **L1154 EN**: Adds a standalone statement or declaration: `InGroup<CXXPre17Compat>, DefaultIgnore;`.
  **L1154 CN**: 添加一条独立语句或声明：`InGroup<CXXPre17Compat>, DefaultIgnore;`。
- **L1155 EN**: Declares TableGen def record `ext_constexpr_on_lambda_cxx17`.
  **L1155 CN**: 声明 TableGen def 记录 `ext_constexpr_on_lambda_cxx17`。
- **L1156 EN**: Adds a standalone statement or declaration: `"'constexpr' on lambda expressions is a C++17 extension">, InGroup<CXX17>;`.
  **L1156 CN**: 添加一条独立语句或声明：`"'constexpr' on lambda expressions is a C++17 extension">, InGroup<CXX17>;`。
- **L1157 EN**: Blank line separating nearby declarations or logic blocks.
  **L1157 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1158 EN**: Comment explains nearby logic, constraints, or intent: `C++20 template lambdas`.
  **L1158 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C++20 template lambdas`。
- **L1159 EN**: Declares TableGen def record `ext_lambda_template_parameter_list`.
  **L1159 CN**: 声明 TableGen def 记录 `ext_lambda_template_parameter_list`。
- **L1160 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"explicit template parameter list for lambdas is a C++20 extension">,`.
  **L1160 CN**: 继续一个多行参数列表、初始化器或聚合项：`"explicit template parameter list for lambdas is a C++20 extension">,`。
- **L1161 EN**: Adds a standalone statement or declaration: `InGroup<CXX20>;`.
  **L1161 CN**: 添加一条独立语句或声明：`InGroup<CXX20>;`。
- **L1162 EN**: Declares TableGen def record `warn_cxx17_compat_lambda_template_parameter_list`.
  **L1162 CN**: 声明 TableGen def 记录 `warn_cxx17_compat_lambda_template_parameter_list`。
- **L1163 EN**: Continues the surrounding expression or declaration: `"explicit template parameter list for lambdas is incompatible with "`.
  **L1163 CN**: 继续构造周围的表达式或声明：`"explicit template parameter list for lambdas is incompatible with "`。
- **L1164 EN**: Adds a standalone statement or declaration: `"C++ standards before C++20">, InGroup<CXXPre20Compat>, DefaultIgnore;`.
  **L1164 CN**: 添加一条独立语句或声明：`"C++ standards before C++20">, InGroup<CXXPre20Compat>, DefaultIgnore;`。
- **L1165 EN**: Declares TableGen def record `err_lambda_template_parameter_list_empty`.
  **L1165 CN**: 声明 TableGen def 记录 `err_lambda_template_parameter_list_empty`。
- **L1166 EN**: Adds a standalone statement or declaration: `"lambda template parameter list cannot be empty">;`.
  **L1166 CN**: 添加一条独立语句或声明：`"lambda template parameter list cannot be empty">;`。
- **L1167 EN**: Blank line separating nearby declarations or logic blocks.
  **L1167 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1168 EN**: Comment explains nearby logic, constraints, or intent: `C++23 static lambdas`.
  **L1168 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C++23 static lambdas`。
- **L1169 EN**: Declares TableGen def record `err_static_lambda`.
  **L1169 CN**: 声明 TableGen def 记录 `err_static_lambda`。
- **L1170 EN**: Adds a standalone statement or declaration: `"static lambdas are a C++23 extension">, InGroup<CXX23>;`.
  **L1170 CN**: 添加一条独立语句或声明：`"static lambdas are a C++23 extension">, InGroup<CXX23>;`。
- **L1171 EN**: Declares TableGen def record `warn_cxx20_compat_static_lambda`.
  **L1171 CN**: 声明 TableGen def 记录 `warn_cxx20_compat_static_lambda`。
- **L1172 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"static lambdas are incompatible with C++ standards before C++23">,`.
  **L1172 CN**: 继续一个多行参数列表、初始化器或聚合项：`"static lambdas are incompatible with C++ standards before C++23">,`。
- **L1173 EN**: Adds a standalone statement or declaration: `InGroup<CXXPre23Compat>, DefaultIgnore;`.
  **L1173 CN**: 添加一条独立语句或声明：`InGroup<CXXPre23Compat>, DefaultIgnore;`。
- **L1174 EN**: Declares TableGen def record `err_static_mutable_lambda`.
  **L1174 CN**: 声明 TableGen def 记录 `err_static_mutable_lambda`。
- **L1175 EN**: Adds a standalone statement or declaration: `"lambda cannot be both mutable and static">;`.
  **L1175 CN**: 添加一条独立语句或声明：`"lambda cannot be both mutable and static">;`。
- **L1176 EN**: Declares TableGen def record `err_static_lambda_captures`.
  **L1176 CN**: 声明 TableGen def 记录 `err_static_lambda_captures`。

### Lines 1177-1200

````tablegen
  "a static lambda cannot have any captures">;

// Availability attribute
def err_expected_version : Error<
  "expected a version of the form 'major[.minor[.subminor]]'">;
def warn_expected_consistent_version_separator : Warning<
  "use same version number separators '_' or '.'; as in "
  "'major[.minor[.subminor]]'">, InGroup<Availability>;
def err_zero_version : Error<
  "version number must have non-zero major, minor, or sub-minor version">;
def err_availability_expected_platform : Error<
  "expected a platform name, e.g., 'macos'">;
def err_availability_expected_environment : Error<
  "expected an environment name, e.g., 'compute'">;

// objc_bridge_related attribute
def err_objcbridge_related_expected_related_class : Error<
  "expected a related Objective-C class name, e.g., 'NSColor'">;
def err_objcbridge_related_selector_name : Error<
  "expected a class method selector with single argument, e.g., 'colorWithCGColor:'">;

def err_availability_expected_change : Error<
  "expected 'introduced', 'deprecated', or 'obsoleted'">;
def err_availability_unknown_change : Error<
````
- **L1177 EN**: Adds a standalone statement or declaration: `"a static lambda cannot have any captures">;`.
  **L1177 CN**: 添加一条独立语句或声明：`"a static lambda cannot have any captures">;`。
- **L1178 EN**: Blank line separating nearby declarations or logic blocks.
  **L1178 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1179 EN**: Comment explains nearby logic, constraints, or intent: `Availability attribute`.
  **L1179 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Availability attribute`。
- **L1180 EN**: Declares TableGen def record `err_expected_version`.
  **L1180 CN**: 声明 TableGen def 记录 `err_expected_version`。
- **L1181 EN**: Adds a standalone statement or declaration: `"expected a version of the form 'major[.minor[.subminor]]'">;`.
  **L1181 CN**: 添加一条独立语句或声明：`"expected a version of the form 'major[.minor[.subminor]]'">;`。
- **L1182 EN**: Declares TableGen def record `warn_expected_consistent_version_separator`.
  **L1182 CN**: 声明 TableGen def 记录 `warn_expected_consistent_version_separator`。
- **L1183 EN**: Continues the surrounding expression or declaration: `"use same version number separators '_' or '.'; as in "`.
  **L1183 CN**: 继续构造周围的表达式或声明：`"use same version number separators '_' or '.'; as in "`。
- **L1184 EN**: Adds a standalone statement or declaration: `"'major[.minor[.subminor]]'">, InGroup<Availability>;`.
  **L1184 CN**: 添加一条独立语句或声明：`"'major[.minor[.subminor]]'">, InGroup<Availability>;`。
- **L1185 EN**: Declares TableGen def record `err_zero_version`.
  **L1185 CN**: 声明 TableGen def 记录 `err_zero_version`。
- **L1186 EN**: Adds a standalone statement or declaration: `"version number must have non-zero major, minor, or sub-minor version">;`.
  **L1186 CN**: 添加一条独立语句或声明：`"version number must have non-zero major, minor, or sub-minor version">;`。
- **L1187 EN**: Declares TableGen def record `err_availability_expected_platform`.
  **L1187 CN**: 声明 TableGen def 记录 `err_availability_expected_platform`。
- **L1188 EN**: Adds a standalone statement or declaration: `"expected a platform name, e.g., 'macos'">;`.
  **L1188 CN**: 添加一条独立语句或声明：`"expected a platform name, e.g., 'macos'">;`。
- **L1189 EN**: Declares TableGen def record `err_availability_expected_environment`.
  **L1189 CN**: 声明 TableGen def 记录 `err_availability_expected_environment`。
- **L1190 EN**: Adds a standalone statement or declaration: `"expected an environment name, e.g., 'compute'">;`.
  **L1190 CN**: 添加一条独立语句或声明：`"expected an environment name, e.g., 'compute'">;`。
- **L1191 EN**: Blank line separating nearby declarations or logic blocks.
  **L1191 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1192 EN**: Comment explains nearby logic, constraints, or intent: `objc_bridge_related attribute`.
  **L1192 CN**: 注释解释附近代码的逻辑、约束或设计意图：`objc_bridge_related attribute`。
- **L1193 EN**: Declares TableGen def record `err_objcbridge_related_expected_related_class`.
  **L1193 CN**: 声明 TableGen def 记录 `err_objcbridge_related_expected_related_class`。
- **L1194 EN**: Adds a standalone statement or declaration: `"expected a related Objective-C class name, e.g., 'NSColor'">;`.
  **L1194 CN**: 添加一条独立语句或声明：`"expected a related Objective-C class name, e.g., 'NSColor'">;`。
- **L1195 EN**: Declares TableGen def record `err_objcbridge_related_selector_name`.
  **L1195 CN**: 声明 TableGen def 记录 `err_objcbridge_related_selector_name`。
- **L1196 EN**: Adds a standalone statement or declaration: `"expected a class method selector with single argument, e.g., 'colorWithCGColor:'">;`.
  **L1196 CN**: 添加一条独立语句或声明：`"expected a class method selector with single argument, e.g., 'colorWithCGColor:'">;`。
- **L1197 EN**: Blank line separating nearby declarations or logic blocks.
  **L1197 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1198 EN**: Declares TableGen def record `err_availability_expected_change`.
  **L1198 CN**: 声明 TableGen def 记录 `err_availability_expected_change`。
- **L1199 EN**: Adds a standalone statement or declaration: `"expected 'introduced', 'deprecated', or 'obsoleted'">;`.
  **L1199 CN**: 添加一条独立语句或声明：`"expected 'introduced', 'deprecated', or 'obsoleted'">;`。
- **L1200 EN**: Declares TableGen def record `err_availability_unknown_change`.
  **L1200 CN**: 声明 TableGen def 记录 `err_availability_unknown_change`。

### Lines 1201-1224

````tablegen
  "%0 is not an availability stage; use 'introduced', 'deprecated', or "
  "'obsoleted'">;
def err_availability_redundant : Error<
  "redundant %0 availability change; only the last specified change will "
  "be used">;
def warn_availability_and_unavailable : Warning<
  "'unavailable' availability overrides all other availability information">,
  InGroup<Availability>;

// @available(...)
def err_avail_query_expected_platform_name : Error<
  "expected a platform name here">;

def err_avail_query_unrecognized_platform_name : Error<
  "unrecognized platform name %0">;
def err_avail_query_anyappleos_min_version : Error<
  "invalid anyAppleOS version '%0' in availability check">;
def err_availability_query_wildcard_required: Error<
  "must handle potential future platforms with '*'">;
def err_availability_query_repeated_platform: Error<
  "version for '%0' already specified">;
def err_availability_query_repeated_star : Error<
  "'*' query has already been specified">;

````
- **L1201 EN**: Continues the surrounding expression or declaration: `"%0 is not an availability stage; use 'introduced', 'deprecated', or "`.
  **L1201 CN**: 继续构造周围的表达式或声明：`"%0 is not an availability stage; use 'introduced', 'deprecated', or "`。
- **L1202 EN**: Adds a standalone statement or declaration: `"'obsoleted'">;`.
  **L1202 CN**: 添加一条独立语句或声明：`"'obsoleted'">;`。
- **L1203 EN**: Declares TableGen def record `err_availability_redundant`.
  **L1203 CN**: 声明 TableGen def 记录 `err_availability_redundant`。
- **L1204 EN**: Continues the surrounding expression or declaration: `"redundant %0 availability change; only the last specified change will "`.
  **L1204 CN**: 继续构造周围的表达式或声明：`"redundant %0 availability change; only the last specified change will "`。
- **L1205 EN**: Adds a standalone statement or declaration: `"be used">;`.
  **L1205 CN**: 添加一条独立语句或声明：`"be used">;`。
- **L1206 EN**: Declares TableGen def record `warn_availability_and_unavailable`.
  **L1206 CN**: 声明 TableGen def 记录 `warn_availability_and_unavailable`。
- **L1207 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'unavailable' availability overrides all other availability information">,`.
  **L1207 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'unavailable' availability overrides all other availability information">,`。
- **L1208 EN**: Adds a standalone statement or declaration: `InGroup<Availability>;`.
  **L1208 CN**: 添加一条独立语句或声明：`InGroup<Availability>;`。
- **L1209 EN**: Blank line separating nearby declarations or logic blocks.
  **L1209 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1210 EN**: Comment explains nearby logic, constraints, or intent: `@available(...)`.
  **L1210 CN**: 注释解释附近代码的逻辑、约束或设计意图：`@available(...)`。
- **L1211 EN**: Declares TableGen def record `err_avail_query_expected_platform_name`.
  **L1211 CN**: 声明 TableGen def 记录 `err_avail_query_expected_platform_name`。
- **L1212 EN**: Adds a standalone statement or declaration: `"expected a platform name here">;`.
  **L1212 CN**: 添加一条独立语句或声明：`"expected a platform name here">;`。
- **L1213 EN**: Blank line separating nearby declarations or logic blocks.
  **L1213 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1214 EN**: Declares TableGen def record `err_avail_query_unrecognized_platform_name`.
  **L1214 CN**: 声明 TableGen def 记录 `err_avail_query_unrecognized_platform_name`。
- **L1215 EN**: Adds a standalone statement or declaration: `"unrecognized platform name %0">;`.
  **L1215 CN**: 添加一条独立语句或声明：`"unrecognized platform name %0">;`。
- **L1216 EN**: Declares TableGen def record `err_avail_query_anyappleos_min_version`.
  **L1216 CN**: 声明 TableGen def 记录 `err_avail_query_anyappleos_min_version`。
- **L1217 EN**: Adds a standalone statement or declaration: `"invalid anyAppleOS version '%0' in availability check">;`.
  **L1217 CN**: 添加一条独立语句或声明：`"invalid anyAppleOS version '%0' in availability check">;`。
- **L1218 EN**: Declares TableGen def record `err_availability_query_wildcard_required`.
  **L1218 CN**: 声明 TableGen def 记录 `err_availability_query_wildcard_required`。
- **L1219 EN**: Adds a standalone statement or declaration: `"must handle potential future platforms with '*'">;`.
  **L1219 CN**: 添加一条独立语句或声明：`"must handle potential future platforms with '*'">;`。
- **L1220 EN**: Declares TableGen def record `err_availability_query_repeated_platform`.
  **L1220 CN**: 声明 TableGen def 记录 `err_availability_query_repeated_platform`。
- **L1221 EN**: Adds a standalone statement or declaration: `"version for '%0' already specified">;`.
  **L1221 CN**: 添加一条独立语句或声明：`"version for '%0' already specified">;`。
- **L1222 EN**: Declares TableGen def record `err_availability_query_repeated_star`.
  **L1222 CN**: 声明 TableGen def 记录 `err_availability_query_repeated_star`。
- **L1223 EN**: Adds a standalone statement or declaration: `"'*' query has already been specified">;`.
  **L1223 CN**: 添加一条独立语句或声明：`"'*' query has already been specified">;`。
- **L1224 EN**: Blank line separating nearby declarations or logic blocks.
  **L1224 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1225-1248

````tablegen
// External source symbol attribute
def err_external_source_symbol_expected_keyword : Error<
  "expected 'language', 'defined_in', 'generated_declaration', or 'USR'">;
def err_external_source_symbol_duplicate_clause : Error<
  "duplicate %0 clause in an 'external_source_symbol' attribute">;

// Type safety attributes
def err_type_safety_unknown_flag : Error<
  "invalid comparison flag %0; use 'layout_compatible' or 'must_be_null'">;

// Type traits
def err_type_trait_arity : Error<
  "type trait requires %0%select{| or more}1 argument%select{|s}2; have "
  "%3 argument%s3">;

// Language specific pragmas
// - Generic warnings
def warn_pragma_expected_lparen : Warning<
  "missing '(' after '#pragma %0' - ignoring">, InGroup<IgnoredPragmas>;
def warn_pragma_expected_rparen : Warning<
  "missing ')' after '#pragma %0' - ignoring">, InGroup<IgnoredPragmas>;
def warn_pragma_expected_identifier : Warning<
  "expected identifier in '#pragma %0' - ignored">, InGroup<IgnoredPragmas>;
def warn_pragma_expected_string : Warning<
````
- **L1225 EN**: Comment explains nearby logic, constraints, or intent: `External source symbol attribute`.
  **L1225 CN**: 注释解释附近代码的逻辑、约束或设计意图：`External source symbol attribute`。
- **L1226 EN**: Declares TableGen def record `err_external_source_symbol_expected_keyword`.
  **L1226 CN**: 声明 TableGen def 记录 `err_external_source_symbol_expected_keyword`。
- **L1227 EN**: Adds a standalone statement or declaration: `"expected 'language', 'defined_in', 'generated_declaration', or 'USR'">;`.
  **L1227 CN**: 添加一条独立语句或声明：`"expected 'language', 'defined_in', 'generated_declaration', or 'USR'">;`。
- **L1228 EN**: Declares TableGen def record `err_external_source_symbol_duplicate_clause`.
  **L1228 CN**: 声明 TableGen def 记录 `err_external_source_symbol_duplicate_clause`。
- **L1229 EN**: Adds a standalone statement or declaration: `"duplicate %0 clause in an 'external_source_symbol' attribute">;`.
  **L1229 CN**: 添加一条独立语句或声明：`"duplicate %0 clause in an 'external_source_symbol' attribute">;`。
- **L1230 EN**: Blank line separating nearby declarations or logic blocks.
  **L1230 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1231 EN**: Comment explains nearby logic, constraints, or intent: `Type safety attributes`.
  **L1231 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Type safety attributes`。
- **L1232 EN**: Declares TableGen def record `err_type_safety_unknown_flag`.
  **L1232 CN**: 声明 TableGen def 记录 `err_type_safety_unknown_flag`。
- **L1233 EN**: Adds a standalone statement or declaration: `"invalid comparison flag %0; use 'layout_compatible' or 'must_be_null'">;`.
  **L1233 CN**: 添加一条独立语句或声明：`"invalid comparison flag %0; use 'layout_compatible' or 'must_be_null'">;`。
- **L1234 EN**: Blank line separating nearby declarations or logic blocks.
  **L1234 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1235 EN**: Comment explains nearby logic, constraints, or intent: `Type traits`.
  **L1235 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Type traits`。
- **L1236 EN**: Declares TableGen def record `err_type_trait_arity`.
  **L1236 CN**: 声明 TableGen def 记录 `err_type_trait_arity`。
- **L1237 EN**: Continues the surrounding expression or declaration: `"type trait requires %0%select{| or more}1 argument%select{|s}2; have "`.
  **L1237 CN**: 继续构造周围的表达式或声明：`"type trait requires %0%select{| or more}1 argument%select{|s}2; have "`。
- **L1238 EN**: Adds a standalone statement or declaration: `"%3 argument%s3">;`.
  **L1238 CN**: 添加一条独立语句或声明：`"%3 argument%s3">;`。
- **L1239 EN**: Blank line separating nearby declarations or logic blocks.
  **L1239 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1240 EN**: Comment explains nearby logic, constraints, or intent: `Language specific pragmas`.
  **L1240 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Language specific pragmas`。
- **L1241 EN**: Comment explains nearby logic, constraints, or intent: `Generic warnings`.
  **L1241 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Generic warnings`。
- **L1242 EN**: Declares TableGen def record `warn_pragma_expected_lparen`.
  **L1242 CN**: 声明 TableGen def 记录 `warn_pragma_expected_lparen`。
- **L1243 EN**: Executes a call or declaration centered on `'`.
  **L1243 CN**: 执行以 `'` 为核心的调用或声明。
- **L1244 EN**: Declares TableGen def record `warn_pragma_expected_rparen`.
  **L1244 CN**: 声明 TableGen def 记录 `warn_pragma_expected_rparen`。
- **L1245 EN**: Adds a standalone statement or declaration: `"missing ')' after '#pragma %0' - ignoring">, InGroup<IgnoredPragmas>;`.
  **L1245 CN**: 添加一条独立语句或声明：`"missing ')' after '#pragma %0' - ignoring">, InGroup<IgnoredPragmas>;`。
- **L1246 EN**: Declares TableGen def record `warn_pragma_expected_identifier`.
  **L1246 CN**: 声明 TableGen def 记录 `warn_pragma_expected_identifier`。
- **L1247 EN**: Adds a standalone statement or declaration: `"expected identifier in '#pragma %0' - ignored">, InGroup<IgnoredPragmas>;`.
  **L1247 CN**: 添加一条独立语句或声明：`"expected identifier in '#pragma %0' - ignored">, InGroup<IgnoredPragmas>;`。
- **L1248 EN**: Declares TableGen def record `warn_pragma_expected_string`.
  **L1248 CN**: 声明 TableGen def 记录 `warn_pragma_expected_string`。

### Lines 1249-1272

````tablegen
  "expected string literal in '#pragma %0' - ignoring">, InGroup<IgnoredPragmas>;
def warn_pragma_missing_argument : Warning<
  "missing argument to '#pragma %0'%select{|; expected %2}1">, InGroup<IgnoredPragmas>;
def warn_pragma_invalid_argument : Warning<
  "unexpected argument '%0' to '#pragma %1'%select{|; expected %3}2">, InGroup<IgnoredPragmas>;

def err_pragma_misplaced_in_decl : Error<"this pragma cannot appear in %0 declaration">;

// '#pragma clang section' related errors
def err_pragma_expected_clang_section_name : Error<
  "expected one of [bss|data|rodata|text|relro] section kind in '#pragma %0'">;
def err_pragma_clang_section_expected_equal : Error<
  "expected '=' following '#pragma clang section %select{invalid|bss|data|rodata|text|relro}0'">;
def warn_pragma_expected_section_name : Warning<
  "expected a string literal for the section name in '#pragma %0' - ignored">,
  InGroup<IgnoredPragmas>;

def warn_pragma_expected_section_push_pop_or_name : Warning<
  "expected push, pop or a string literal for the section name in '#pragma %0' - ignored">,
  InGroup<IgnoredPragmas>;
def warn_pragma_expected_section_label_or_name : Warning<
  "expected a stack label or a string literal for the section name in '#pragma %0' - ignored">,
  InGroup<IgnoredPragmas>;
def warn_pragma_expected_init_seg : Warning<
````
- **L1249 EN**: Adds a standalone statement or declaration: `"expected string literal in '#pragma %0' - ignoring">, InGroup<IgnoredPragmas>;`.
  **L1249 CN**: 添加一条独立语句或声明：`"expected string literal in '#pragma %0' - ignoring">, InGroup<IgnoredPragmas>;`。
- **L1250 EN**: Declares TableGen def record `warn_pragma_missing_argument`.
  **L1250 CN**: 声明 TableGen def 记录 `warn_pragma_missing_argument`。
- **L1251 EN**: Adds a standalone statement or declaration: `"missing argument to '#pragma %0'%select{|; expected %2}1">, InGroup<IgnoredPragmas>;`.
  **L1251 CN**: 添加一条独立语句或声明：`"missing argument to '#pragma %0'%select{|; expected %2}1">, InGroup<IgnoredPragmas>;`。
- **L1252 EN**: Declares TableGen def record `warn_pragma_invalid_argument`.
  **L1252 CN**: 声明 TableGen def 记录 `warn_pragma_invalid_argument`。
- **L1253 EN**: Adds a standalone statement or declaration: `"unexpected argument '%0' to '#pragma %1'%select{|; expected %3}2">, InGroup<IgnoredPragmas>;`.
  **L1253 CN**: 添加一条独立语句或声明：`"unexpected argument '%0' to '#pragma %1'%select{|; expected %3}2">, InGroup<IgnoredPragmas>;`。
- **L1254 EN**: Blank line separating nearby declarations or logic blocks.
  **L1254 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1255 EN**: Declares TableGen def record `err_pragma_misplaced_in_decl`.
  **L1255 CN**: 声明 TableGen def 记录 `err_pragma_misplaced_in_decl`。
- **L1256 EN**: Blank line separating nearby declarations or logic blocks.
  **L1256 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1257 EN**: Comment explains nearby logic, constraints, or intent: `'#pragma clang section' related errors`.
  **L1257 CN**: 注释解释附近代码的逻辑、约束或设计意图：`'#pragma clang section' related errors`。
- **L1258 EN**: Declares TableGen def record `err_pragma_expected_clang_section_name`.
  **L1258 CN**: 声明 TableGen def 记录 `err_pragma_expected_clang_section_name`。
- **L1259 EN**: Adds a standalone statement or declaration: `"expected one of [bss|data|rodata|text|relro] section kind in '#pragma %0'">;`.
  **L1259 CN**: 添加一条独立语句或声明：`"expected one of [bss|data|rodata|text|relro] section kind in '#pragma %0'">;`。
- **L1260 EN**: Declares TableGen def record `err_pragma_clang_section_expected_equal`.
  **L1260 CN**: 声明 TableGen def 记录 `err_pragma_clang_section_expected_equal`。
- **L1261 EN**: Adds a standalone statement or declaration: `"expected '=' following '#pragma clang section %select{invalid|bss|data|rodata|text|relro}0'">;`.
  **L1261 CN**: 添加一条独立语句或声明：`"expected '=' following '#pragma clang section %select{invalid|bss|data|rodata|text|relro}0'">;`。
- **L1262 EN**: Declares TableGen def record `warn_pragma_expected_section_name`.
  **L1262 CN**: 声明 TableGen def 记录 `warn_pragma_expected_section_name`。
- **L1263 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"expected a string literal for the section name in '#pragma %0' - ignored">,`.
  **L1263 CN**: 继续一个多行参数列表、初始化器或聚合项：`"expected a string literal for the section name in '#pragma %0' - ignored">,`。
- **L1264 EN**: Adds a standalone statement or declaration: `InGroup<IgnoredPragmas>;`.
  **L1264 CN**: 添加一条独立语句或声明：`InGroup<IgnoredPragmas>;`。
- **L1265 EN**: Blank line separating nearby declarations or logic blocks.
  **L1265 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1266 EN**: Declares TableGen def record `warn_pragma_expected_section_push_pop_or_name`.
  **L1266 CN**: 声明 TableGen def 记录 `warn_pragma_expected_section_push_pop_or_name`。
- **L1267 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"expected push, pop or a string literal for the section name in '#pragma %0' - ignored">,`.
  **L1267 CN**: 继续一个多行参数列表、初始化器或聚合项：`"expected push, pop or a string literal for the section name in '#pragma %0' - ignored">,`。
- **L1268 EN**: Adds a standalone statement or declaration: `InGroup<IgnoredPragmas>;`.
  **L1268 CN**: 添加一条独立语句或声明：`InGroup<IgnoredPragmas>;`。
- **L1269 EN**: Declares TableGen def record `warn_pragma_expected_section_label_or_name`.
  **L1269 CN**: 声明 TableGen def 记录 `warn_pragma_expected_section_label_or_name`。
- **L1270 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"expected a stack label or a string literal for the section name in '#pragma %0' - ignored">,`.
  **L1270 CN**: 继续一个多行参数列表、初始化器或聚合项：`"expected a stack label or a string literal for the section name in '#pragma %0' - ignored">,`。
- **L1271 EN**: Adds a standalone statement or declaration: `InGroup<IgnoredPragmas>;`.
  **L1271 CN**: 添加一条独立语句或声明：`InGroup<IgnoredPragmas>;`。
- **L1272 EN**: Declares TableGen def record `warn_pragma_expected_init_seg`.
  **L1272 CN**: 声明 TableGen def 记录 `warn_pragma_expected_init_seg`。

### Lines 1273-1296

````tablegen
  "expected 'compiler', 'lib', 'user', or a string literal for the section name in '#pragma %0' - ignored">,
  InGroup<IgnoredPragmas>;

def err_pragma_expected_integer : Error<"expected an integer argument in '#pragma %0'">;
def warn_pragma_expected_integer : Warning<
  "expected integer between %0 and %1 inclusive in '#pragma %2' - ignored">,
  InGroup<IgnoredPragmas>;
def warn_pragma_ms_struct : Warning<
  "incorrect use of '#pragma ms_struct on|off' - ignored">,
  InGroup<IgnoredPragmas>;
def warn_pragma_ms_fenv_access : Warning<
  "incorrect use of '#pragma fenv_access (on|off)' - ignored">,
  InGroup<IgnoredPragmas>;
def warn_pragma_extra_tokens_at_eol : Warning<
  "extra tokens at end of '#pragma %0' - ignored">,
  InGroup<IgnoredPragmas>;
def warn_pragma_expected_comma : Warning<
  "expected ',' in '#pragma %0'">, InGroup<IgnoredPragmas>;
def warn_pragma_expected_punc : Warning<
  "expected ')' or ',' in '#pragma %0'">, InGroup<IgnoredPragmas>;
def warn_pragma_expected_non_wide_string : Warning<
  "expected non-wide string literal in '#pragma %0'">, InGroup<IgnoredPragmas>;
// - Generic errors
def err_pragma_missing_argument : Error<
````
- **L1273 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"expected 'compiler', 'lib', 'user', or a string literal for the section name in '#pragma %0' - ignored">,`.
  **L1273 CN**: 继续一个多行参数列表、初始化器或聚合项：`"expected 'compiler', 'lib', 'user', or a string literal for the section name in '#pragma %0' - ignored">,`。
- **L1274 EN**: Adds a standalone statement or declaration: `InGroup<IgnoredPragmas>;`.
  **L1274 CN**: 添加一条独立语句或声明：`InGroup<IgnoredPragmas>;`。
- **L1275 EN**: Blank line separating nearby declarations or logic blocks.
  **L1275 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1276 EN**: Declares TableGen def record `err_pragma_expected_integer`.
  **L1276 CN**: 声明 TableGen def 记录 `err_pragma_expected_integer`。
- **L1277 EN**: Declares TableGen def record `warn_pragma_expected_integer`.
  **L1277 CN**: 声明 TableGen def 记录 `warn_pragma_expected_integer`。
- **L1278 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"expected integer between %0 and %1 inclusive in '#pragma %2' - ignored">,`.
  **L1278 CN**: 继续一个多行参数列表、初始化器或聚合项：`"expected integer between %0 and %1 inclusive in '#pragma %2' - ignored">,`。
- **L1279 EN**: Adds a standalone statement or declaration: `InGroup<IgnoredPragmas>;`.
  **L1279 CN**: 添加一条独立语句或声明：`InGroup<IgnoredPragmas>;`。
- **L1280 EN**: Declares TableGen def record `warn_pragma_ms_struct`.
  **L1280 CN**: 声明 TableGen def 记录 `warn_pragma_ms_struct`。
- **L1281 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"incorrect use of '#pragma ms_struct on|off' - ignored">,`.
  **L1281 CN**: 继续一个多行参数列表、初始化器或聚合项：`"incorrect use of '#pragma ms_struct on|off' - ignored">,`。
- **L1282 EN**: Adds a standalone statement or declaration: `InGroup<IgnoredPragmas>;`.
  **L1282 CN**: 添加一条独立语句或声明：`InGroup<IgnoredPragmas>;`。
- **L1283 EN**: Declares TableGen def record `warn_pragma_ms_fenv_access`.
  **L1283 CN**: 声明 TableGen def 记录 `warn_pragma_ms_fenv_access`。
- **L1284 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"incorrect use of '#pragma fenv_access (on|off)' - ignored">,`.
  **L1284 CN**: 继续一个多行参数列表、初始化器或聚合项：`"incorrect use of '#pragma fenv_access (on|off)' - ignored">,`。
- **L1285 EN**: Adds a standalone statement or declaration: `InGroup<IgnoredPragmas>;`.
  **L1285 CN**: 添加一条独立语句或声明：`InGroup<IgnoredPragmas>;`。
- **L1286 EN**: Declares TableGen def record `warn_pragma_extra_tokens_at_eol`.
  **L1286 CN**: 声明 TableGen def 记录 `warn_pragma_extra_tokens_at_eol`。
- **L1287 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"extra tokens at end of '#pragma %0' - ignored">,`.
  **L1287 CN**: 继续一个多行参数列表、初始化器或聚合项：`"extra tokens at end of '#pragma %0' - ignored">,`。
- **L1288 EN**: Adds a standalone statement or declaration: `InGroup<IgnoredPragmas>;`.
  **L1288 CN**: 添加一条独立语句或声明：`InGroup<IgnoredPragmas>;`。
- **L1289 EN**: Declares TableGen def record `warn_pragma_expected_comma`.
  **L1289 CN**: 声明 TableGen def 记录 `warn_pragma_expected_comma`。
- **L1290 EN**: Adds a standalone statement or declaration: `"expected ',' in '#pragma %0'">, InGroup<IgnoredPragmas>;`.
  **L1290 CN**: 添加一条独立语句或声明：`"expected ',' in '#pragma %0'">, InGroup<IgnoredPragmas>;`。
- **L1291 EN**: Declares TableGen def record `warn_pragma_expected_punc`.
  **L1291 CN**: 声明 TableGen def 记录 `warn_pragma_expected_punc`。
- **L1292 EN**: Adds a standalone statement or declaration: `"expected ')' or ',' in '#pragma %0'">, InGroup<IgnoredPragmas>;`.
  **L1292 CN**: 添加一条独立语句或声明：`"expected ')' or ',' in '#pragma %0'">, InGroup<IgnoredPragmas>;`。
- **L1293 EN**: Declares TableGen def record `warn_pragma_expected_non_wide_string`.
  **L1293 CN**: 声明 TableGen def 记录 `warn_pragma_expected_non_wide_string`。
- **L1294 EN**: Adds a standalone statement or declaration: `"expected non-wide string literal in '#pragma %0'">, InGroup<IgnoredPragmas>;`.
  **L1294 CN**: 添加一条独立语句或声明：`"expected non-wide string literal in '#pragma %0'">, InGroup<IgnoredPragmas>;`。
- **L1295 EN**: Comment explains nearby logic, constraints, or intent: `Generic errors`.
  **L1295 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Generic errors`。
- **L1296 EN**: Declares TableGen def record `err_pragma_missing_argument`.
  **L1296 CN**: 声明 TableGen def 记录 `err_pragma_missing_argument`。

### Lines 1297-1320

````tablegen
  "missing argument to '#pragma %0'%select{|; expected %2}1">;
// - #pragma options
def warn_pragma_options_expected_align : Warning<
  "expected 'align' following '#pragma options' - ignored">,
  InGroup<IgnoredPragmas>;
def warn_pragma_align_expected_equal : Warning<
  "expected '=' following '#pragma %select{align|options align}0' - ignored">,
  InGroup<IgnoredPragmas>;
def warn_pragma_align_invalid_option : Warning<
  "invalid alignment option in '#pragma %select{align|options align}0' - ignored">,
  InGroup<IgnoredPragmas>;
// - #pragma pack
def warn_pragma_unsupported_action : Warning<
  "known but unsupported action '%1' for '#pragma %0' - ignored">,
  InGroup<IgnoredPragmas>;
def warn_pragma_invalid_specific_action : Warning<
  "unknown action '%1' for '#pragma %0' - ignored">,
  InGroup<IgnoredPragmas>;
def warn_pragma_expected_action_or_r_paren : Warning<
  "expected action or ')' in '#pragma %0' - ignored">,
  InGroup<IgnoredPragmas>;
def warn_pragma_invalid_action : Warning<
  "unknown action for '#pragma %0' - ignored">,
  InGroup<IgnoredPragmas>;
````
- **L1297 EN**: Adds a standalone statement or declaration: `"missing argument to '#pragma %0'%select{|; expected %2}1">;`.
  **L1297 CN**: 添加一条独立语句或声明：`"missing argument to '#pragma %0'%select{|; expected %2}1">;`。
- **L1298 EN**: Comment explains nearby logic, constraints, or intent: `#pragma options`.
  **L1298 CN**: 注释解释附近代码的逻辑、约束或设计意图：`#pragma options`。
- **L1299 EN**: Declares TableGen def record `warn_pragma_options_expected_align`.
  **L1299 CN**: 声明 TableGen def 记录 `warn_pragma_options_expected_align`。
- **L1300 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"expected 'align' following '#pragma options' - ignored">,`.
  **L1300 CN**: 继续一个多行参数列表、初始化器或聚合项：`"expected 'align' following '#pragma options' - ignored">,`。
- **L1301 EN**: Adds a standalone statement or declaration: `InGroup<IgnoredPragmas>;`.
  **L1301 CN**: 添加一条独立语句或声明：`InGroup<IgnoredPragmas>;`。
- **L1302 EN**: Declares TableGen def record `warn_pragma_align_expected_equal`.
  **L1302 CN**: 声明 TableGen def 记录 `warn_pragma_align_expected_equal`。
- **L1303 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"expected '=' following '#pragma %select{align|options align}0' - ignored">,`.
  **L1303 CN**: 继续一个多行参数列表、初始化器或聚合项：`"expected '=' following '#pragma %select{align|options align}0' - ignored">,`。
- **L1304 EN**: Adds a standalone statement or declaration: `InGroup<IgnoredPragmas>;`.
  **L1304 CN**: 添加一条独立语句或声明：`InGroup<IgnoredPragmas>;`。
- **L1305 EN**: Declares TableGen def record `warn_pragma_align_invalid_option`.
  **L1305 CN**: 声明 TableGen def 记录 `warn_pragma_align_invalid_option`。
- **L1306 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"invalid alignment option in '#pragma %select{align|options align}0' - ignored">,`.
  **L1306 CN**: 继续一个多行参数列表、初始化器或聚合项：`"invalid alignment option in '#pragma %select{align|options align}0' - ignored">,`。
- **L1307 EN**: Adds a standalone statement or declaration: `InGroup<IgnoredPragmas>;`.
  **L1307 CN**: 添加一条独立语句或声明：`InGroup<IgnoredPragmas>;`。
- **L1308 EN**: Comment explains nearby logic, constraints, or intent: `#pragma pack`.
  **L1308 CN**: 注释解释附近代码的逻辑、约束或设计意图：`#pragma pack`。
- **L1309 EN**: Declares TableGen def record `warn_pragma_unsupported_action`.
  **L1309 CN**: 声明 TableGen def 记录 `warn_pragma_unsupported_action`。
- **L1310 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"known but unsupported action '%1' for '#pragma %0' - ignored">,`.
  **L1310 CN**: 继续一个多行参数列表、初始化器或聚合项：`"known but unsupported action '%1' for '#pragma %0' - ignored">,`。
- **L1311 EN**: Adds a standalone statement or declaration: `InGroup<IgnoredPragmas>;`.
  **L1311 CN**: 添加一条独立语句或声明：`InGroup<IgnoredPragmas>;`。
- **L1312 EN**: Declares TableGen def record `warn_pragma_invalid_specific_action`.
  **L1312 CN**: 声明 TableGen def 记录 `warn_pragma_invalid_specific_action`。
- **L1313 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"unknown action '%1' for '#pragma %0' - ignored">,`.
  **L1313 CN**: 继续一个多行参数列表、初始化器或聚合项：`"unknown action '%1' for '#pragma %0' - ignored">,`。
- **L1314 EN**: Adds a standalone statement or declaration: `InGroup<IgnoredPragmas>;`.
  **L1314 CN**: 添加一条独立语句或声明：`InGroup<IgnoredPragmas>;`。
- **L1315 EN**: Declares TableGen def record `warn_pragma_expected_action_or_r_paren`.
  **L1315 CN**: 声明 TableGen def 记录 `warn_pragma_expected_action_or_r_paren`。
- **L1316 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"expected action or ')' in '#pragma %0' - ignored">,`.
  **L1316 CN**: 继续一个多行参数列表、初始化器或聚合项：`"expected action or ')' in '#pragma %0' - ignored">,`。
- **L1317 EN**: Adds a standalone statement or declaration: `InGroup<IgnoredPragmas>;`.
  **L1317 CN**: 添加一条独立语句或声明：`InGroup<IgnoredPragmas>;`。
- **L1318 EN**: Declares TableGen def record `warn_pragma_invalid_action`.
  **L1318 CN**: 声明 TableGen def 记录 `warn_pragma_invalid_action`。
- **L1319 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"unknown action for '#pragma %0' - ignored">,`.
  **L1319 CN**: 继续一个多行参数列表、初始化器或聚合项：`"unknown action for '#pragma %0' - ignored">,`。
- **L1320 EN**: Adds a standalone statement or declaration: `InGroup<IgnoredPragmas>;`.
  **L1320 CN**: 添加一条独立语句或声明：`InGroup<IgnoredPragmas>;`。

### Lines 1321-1344

````tablegen
def warn_pragma_pack_malformed : Warning<
  "expected integer or identifier in '#pragma pack' - ignored">,
  InGroup<IgnoredPragmas>;
// - #pragma intrinsic
def warn_pragma_intrinsic_builtin : Warning<
  "%0 is not a recognized builtin%select{|; consider including <intrin.h> to access non-builtin intrinsics}1">,
  InGroup<IgnoredPragmaIntrinsic>;
// - #pragma unused
def warn_pragma_unused_expected_var : Warning<
  "expected '#pragma unused' argument to be a variable name">,
  InGroup<IgnoredPragmas>;
// - #pragma init_seg
def warn_pragma_init_seg_unsupported_target : Warning<
  "'#pragma init_seg' is only supported when targeting a "
  "Microsoft environment">,
  InGroup<IgnoredPragmas>;
// - #pragma restricted to file scope or start of compound statement
def err_pragma_file_or_compound_scope : Error<
  "'#pragma %0' can only appear at file scope or at the start of a "
  "compound statement">;
// - #pragma stdc unknown
def ext_stdc_pragma_ignored : ExtWarn<"unknown pragma in STDC namespace">,
   InGroup<UnknownPragmas>;
// The C standard 7.6.1p2 says "The [FENV_ACCESS] pragma shall occur either
````
- **L1321 EN**: Declares TableGen def record `warn_pragma_pack_malformed`.
  **L1321 CN**: 声明 TableGen def 记录 `warn_pragma_pack_malformed`。
- **L1322 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"expected integer or identifier in '#pragma pack' - ignored">,`.
  **L1322 CN**: 继续一个多行参数列表、初始化器或聚合项：`"expected integer or identifier in '#pragma pack' - ignored">,`。
- **L1323 EN**: Adds a standalone statement or declaration: `InGroup<IgnoredPragmas>;`.
  **L1323 CN**: 添加一条独立语句或声明：`InGroup<IgnoredPragmas>;`。
- **L1324 EN**: Comment explains nearby logic, constraints, or intent: `#pragma intrinsic`.
  **L1324 CN**: 注释解释附近代码的逻辑、约束或设计意图：`#pragma intrinsic`。
- **L1325 EN**: Declares TableGen def record `warn_pragma_intrinsic_builtin`.
  **L1325 CN**: 声明 TableGen def 记录 `warn_pragma_intrinsic_builtin`。
- **L1326 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"%0 is not a recognized builtin%select{|; consider including <intrin.h> to access non-builtin intrinsics}1">,`.
  **L1326 CN**: 继续一个多行参数列表、初始化器或聚合项：`"%0 is not a recognized builtin%select{|; consider including <intrin.h> to access non-builtin intrinsics}1">,`。
- **L1327 EN**: Adds a standalone statement or declaration: `InGroup<IgnoredPragmaIntrinsic>;`.
  **L1327 CN**: 添加一条独立语句或声明：`InGroup<IgnoredPragmaIntrinsic>;`。
- **L1328 EN**: Comment explains nearby logic, constraints, or intent: `#pragma unused`.
  **L1328 CN**: 注释解释附近代码的逻辑、约束或设计意图：`#pragma unused`。
- **L1329 EN**: Declares TableGen def record `warn_pragma_unused_expected_var`.
  **L1329 CN**: 声明 TableGen def 记录 `warn_pragma_unused_expected_var`。
- **L1330 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"expected '#pragma unused' argument to be a variable name">,`.
  **L1330 CN**: 继续一个多行参数列表、初始化器或聚合项：`"expected '#pragma unused' argument to be a variable name">,`。
- **L1331 EN**: Adds a standalone statement or declaration: `InGroup<IgnoredPragmas>;`.
  **L1331 CN**: 添加一条独立语句或声明：`InGroup<IgnoredPragmas>;`。
- **L1332 EN**: Comment explains nearby logic, constraints, or intent: `#pragma init_seg`.
  **L1332 CN**: 注释解释附近代码的逻辑、约束或设计意图：`#pragma init_seg`。
- **L1333 EN**: Declares TableGen def record `warn_pragma_init_seg_unsupported_target`.
  **L1333 CN**: 声明 TableGen def 记录 `warn_pragma_init_seg_unsupported_target`。
- **L1334 EN**: Continues the surrounding expression or declaration: `"'#pragma init_seg' is only supported when targeting a "`.
  **L1334 CN**: 继续构造周围的表达式或声明：`"'#pragma init_seg' is only supported when targeting a "`。
- **L1335 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Microsoft environment">,`.
  **L1335 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Microsoft environment">,`。
- **L1336 EN**: Adds a standalone statement or declaration: `InGroup<IgnoredPragmas>;`.
  **L1336 CN**: 添加一条独立语句或声明：`InGroup<IgnoredPragmas>;`。
- **L1337 EN**: Comment explains nearby logic, constraints, or intent: `#pragma restricted to file scope or start of compound statement`.
  **L1337 CN**: 注释解释附近代码的逻辑、约束或设计意图：`#pragma restricted to file scope or start of compound statement`。
- **L1338 EN**: Declares TableGen def record `err_pragma_file_or_compound_scope`.
  **L1338 CN**: 声明 TableGen def 记录 `err_pragma_file_or_compound_scope`。
- **L1339 EN**: Continues the surrounding expression or declaration: `"'#pragma %0' can only appear at file scope or at the start of a "`.
  **L1339 CN**: 继续构造周围的表达式或声明：`"'#pragma %0' can only appear at file scope or at the start of a "`。
- **L1340 EN**: Adds a standalone statement or declaration: `"compound statement">;`.
  **L1340 CN**: 添加一条独立语句或声明：`"compound statement">;`。
- **L1341 EN**: Comment explains nearby logic, constraints, or intent: `#pragma stdc unknown`.
  **L1341 CN**: 注释解释附近代码的逻辑、约束或设计意图：`#pragma stdc unknown`。
- **L1342 EN**: Declares TableGen def record `ext_stdc_pragma_ignored`.
  **L1342 CN**: 声明 TableGen def 记录 `ext_stdc_pragma_ignored`。
- **L1343 EN**: Adds a standalone statement or declaration: `InGroup<UnknownPragmas>;`.
  **L1343 CN**: 添加一条独立语句或声明：`InGroup<UnknownPragmas>;`。
- **L1344 EN**: Comment explains nearby logic, constraints, or intent: `The C standard 7.6.1p2 says "The [FENV_ACCESS] pragma shall occur either`.
  **L1344 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The C standard 7.6.1p2 says "The [FENV_ACCESS] pragma shall occur either`。

### Lines 1345-1368

````tablegen
// outside external declarations or preceding all explicit declarations and
// statements inside a compound statement.
def warn_stdc_fenv_round_not_supported :
   Warning<"pragma STDC FENV_ROUND is not supported">,
   InGroup<UnknownPragmas>;
def warn_stdc_unknown_rounding_mode : Warning<
  "invalid or unsupported rounding mode in '#pragma STDC FENV_ROUND' - ignored">,
  InGroup<IgnoredPragmas>;
def warn_pragma_fp_ignored : Warning<
  "'#pragma %0' is not supported on this target - ignored">,
  InGroup<IgnoredPragmas>;
// - #pragma comment
def err_pragma_comment_malformed : Error<
  "pragma comment requires parenthesized identifier and optional string">;
def err_pragma_comment_unknown_kind : Error<"unknown kind of pragma comment">;
// PS4 recognizes only #pragma comment(lib)
def warn_pragma_comment_ignored : Warning<"'#pragma comment %0' ignored">,
  InGroup<IgnoredPragmas>;
// - #pragma detect_mismatch
def err_pragma_detect_mismatch_malformed : Error<
  "pragma detect_mismatch is malformed; it requires two comma-separated "
  "string literals">;
// - #pragma float_control
def err_pragma_float_control_malformed : Error<
````
- **L1345 EN**: Comment explains nearby logic, constraints, or intent: `outside external declarations or preceding all explicit declarations and`.
  **L1345 CN**: 注释解释附近代码的逻辑、约束或设计意图：`outside external declarations or preceding all explicit declarations and`。
- **L1346 EN**: Comment explains nearby logic, constraints, or intent: `statements inside a compound statement.`.
  **L1346 CN**: 注释解释附近代码的逻辑、约束或设计意图：`statements inside a compound statement.`。
- **L1347 EN**: Declares TableGen def record `warn_stdc_fenv_round_not_supported`.
  **L1347 CN**: 声明 TableGen def 记录 `warn_stdc_fenv_round_not_supported`。
- **L1348 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Warning<"pragma STDC FENV_ROUND is not supported">,`.
  **L1348 CN**: 继续一个多行参数列表、初始化器或聚合项：`Warning<"pragma STDC FENV_ROUND is not supported">,`。
- **L1349 EN**: Adds a standalone statement or declaration: `InGroup<UnknownPragmas>;`.
  **L1349 CN**: 添加一条独立语句或声明：`InGroup<UnknownPragmas>;`。
- **L1350 EN**: Declares TableGen def record `warn_stdc_unknown_rounding_mode`.
  **L1350 CN**: 声明 TableGen def 记录 `warn_stdc_unknown_rounding_mode`。
- **L1351 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"invalid or unsupported rounding mode in '#pragma STDC FENV_ROUND' - ignored">,`.
  **L1351 CN**: 继续一个多行参数列表、初始化器或聚合项：`"invalid or unsupported rounding mode in '#pragma STDC FENV_ROUND' - ignored">,`。
- **L1352 EN**: Adds a standalone statement or declaration: `InGroup<IgnoredPragmas>;`.
  **L1352 CN**: 添加一条独立语句或声明：`InGroup<IgnoredPragmas>;`。
- **L1353 EN**: Declares TableGen def record `warn_pragma_fp_ignored`.
  **L1353 CN**: 声明 TableGen def 记录 `warn_pragma_fp_ignored`。
- **L1354 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'#pragma %0' is not supported on this target - ignored">,`.
  **L1354 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'#pragma %0' is not supported on this target - ignored">,`。
- **L1355 EN**: Adds a standalone statement or declaration: `InGroup<IgnoredPragmas>;`.
  **L1355 CN**: 添加一条独立语句或声明：`InGroup<IgnoredPragmas>;`。
- **L1356 EN**: Comment explains nearby logic, constraints, or intent: `#pragma comment`.
  **L1356 CN**: 注释解释附近代码的逻辑、约束或设计意图：`#pragma comment`。
- **L1357 EN**: Declares TableGen def record `err_pragma_comment_malformed`.
  **L1357 CN**: 声明 TableGen def 记录 `err_pragma_comment_malformed`。
- **L1358 EN**: Adds a standalone statement or declaration: `"pragma comment requires parenthesized identifier and optional string">;`.
  **L1358 CN**: 添加一条独立语句或声明：`"pragma comment requires parenthesized identifier and optional string">;`。
- **L1359 EN**: Declares TableGen def record `err_pragma_comment_unknown_kind`.
  **L1359 CN**: 声明 TableGen def 记录 `err_pragma_comment_unknown_kind`。
- **L1360 EN**: Comment explains nearby logic, constraints, or intent: `PS4 recognizes only #pragma comment(lib)`.
  **L1360 CN**: 注释解释附近代码的逻辑、约束或设计意图：`PS4 recognizes only #pragma comment(lib)`。
- **L1361 EN**: Declares TableGen def record `warn_pragma_comment_ignored`.
  **L1361 CN**: 声明 TableGen def 记录 `warn_pragma_comment_ignored`。
- **L1362 EN**: Adds a standalone statement or declaration: `InGroup<IgnoredPragmas>;`.
  **L1362 CN**: 添加一条独立语句或声明：`InGroup<IgnoredPragmas>;`。
- **L1363 EN**: Comment explains nearby logic, constraints, or intent: `#pragma detect_mismatch`.
  **L1363 CN**: 注释解释附近代码的逻辑、约束或设计意图：`#pragma detect_mismatch`。
- **L1364 EN**: Declares TableGen def record `err_pragma_detect_mismatch_malformed`.
  **L1364 CN**: 声明 TableGen def 记录 `err_pragma_detect_mismatch_malformed`。
- **L1365 EN**: Continues the surrounding expression or declaration: `"pragma detect_mismatch is malformed; it requires two comma-separated "`.
  **L1365 CN**: 继续构造周围的表达式或声明：`"pragma detect_mismatch is malformed; it requires two comma-separated "`。
- **L1366 EN**: Adds a standalone statement or declaration: `"string literals">;`.
  **L1366 CN**: 添加一条独立语句或声明：`"string literals">;`。
- **L1367 EN**: Comment explains nearby logic, constraints, or intent: `#pragma float_control`.
  **L1367 CN**: 注释解释附近代码的逻辑、约束或设计意图：`#pragma float_control`。
- **L1368 EN**: Declares TableGen def record `err_pragma_float_control_malformed`.
  **L1368 CN**: 声明 TableGen def 记录 `err_pragma_float_control_malformed`。

### Lines 1369-1392

````tablegen
  "pragma float_control is malformed; use 'float_control({push|pop})' or "
  "'float_control({precise|except}, {on|off} [,push])'">;
// - #pragma pointers_to_members
def err_pragma_pointers_to_members_unknown_kind : Error<
  "unexpected %0, expected to see one of %select{|'best_case', 'full_generality', }1"
  "'single_inheritance', 'multiple_inheritance', or 'virtual_inheritance'">;
// - #pragma clang optimize on/off
def err_pragma_optimize_invalid_argument : Error<
  "unexpected argument '%0' to '#pragma clang optimize'; "
  "expected 'on' or 'off'">;
def err_pragma_optimize_extra_argument : Error<
  "unexpected extra argument '%0' to '#pragma clang optimize'">;
// - #pragma clang attribute
def err_pragma_attribute_expected_push_pop_paren : Error<
  "expected 'push', 'pop', or '(' after '#pragma clang attribute'">;
def err_pragma_attribute_invalid_argument : Error<
  "unexpected argument '%0' to '#pragma clang attribute'; "
  "expected 'push' or 'pop'">;
def err_pragma_attribute_expected_attribute : Error<
  "expected an attribute after '('">;
def err_pragma_attribute_expected_attribute_name : Error<
  "expected identifier that represents an attribute name">;
def err_pragma_attribute_extra_tokens_after_attribute : Error<
  "extra tokens after attribute in a '#pragma clang attribute push'">;
````
- **L1369 EN**: Continues logic associated with callable symbol `float_control`.
  **L1369 CN**: 继续与可调用符号 `float_control` 相关的逻辑。
- **L1370 EN**: Executes a call or declaration centered on `"'float_control`.
  **L1370 CN**: 执行以 `"'float_control` 为核心的调用或声明。
- **L1371 EN**: Comment explains nearby logic, constraints, or intent: `#pragma pointers_to_members`.
  **L1371 CN**: 注释解释附近代码的逻辑、约束或设计意图：`#pragma pointers_to_members`。
- **L1372 EN**: Declares TableGen def record `err_pragma_pointers_to_members_unknown_kind`.
  **L1372 CN**: 声明 TableGen def 记录 `err_pragma_pointers_to_members_unknown_kind`。
- **L1373 EN**: Continues the surrounding expression or declaration: `"unexpected %0, expected to see one of %select{|'best_case', 'full_generality', }1"`.
  **L1373 CN**: 继续构造周围的表达式或声明：`"unexpected %0, expected to see one of %select{|'best_case', 'full_generality', }1"`。
- **L1374 EN**: Adds a standalone statement or declaration: `"'single_inheritance', 'multiple_inheritance', or 'virtual_inheritance'">;`.
  **L1374 CN**: 添加一条独立语句或声明：`"'single_inheritance', 'multiple_inheritance', or 'virtual_inheritance'">;`。
- **L1375 EN**: Comment explains nearby logic, constraints, or intent: `#pragma clang optimize on/off`.
  **L1375 CN**: 注释解释附近代码的逻辑、约束或设计意图：`#pragma clang optimize on/off`。
- **L1376 EN**: Declares TableGen def record `err_pragma_optimize_invalid_argument`.
  **L1376 CN**: 声明 TableGen def 记录 `err_pragma_optimize_invalid_argument`。
- **L1377 EN**: Continues the surrounding expression or declaration: `"unexpected argument '%0' to '#pragma clang optimize'; "`.
  **L1377 CN**: 继续构造周围的表达式或声明：`"unexpected argument '%0' to '#pragma clang optimize'; "`。
- **L1378 EN**: Adds a standalone statement or declaration: `"expected 'on' or 'off'">;`.
  **L1378 CN**: 添加一条独立语句或声明：`"expected 'on' or 'off'">;`。
- **L1379 EN**: Declares TableGen def record `err_pragma_optimize_extra_argument`.
  **L1379 CN**: 声明 TableGen def 记录 `err_pragma_optimize_extra_argument`。
- **L1380 EN**: Adds a standalone statement or declaration: `"unexpected extra argument '%0' to '#pragma clang optimize'">;`.
  **L1380 CN**: 添加一条独立语句或声明：`"unexpected extra argument '%0' to '#pragma clang optimize'">;`。
- **L1381 EN**: Comment explains nearby logic, constraints, or intent: `#pragma clang attribute`.
  **L1381 CN**: 注释解释附近代码的逻辑、约束或设计意图：`#pragma clang attribute`。
- **L1382 EN**: Declares TableGen def record `err_pragma_attribute_expected_push_pop_paren`.
  **L1382 CN**: 声明 TableGen def 记录 `err_pragma_attribute_expected_push_pop_paren`。
- **L1383 EN**: Executes a call or declaration centered on `'`.
  **L1383 CN**: 执行以 `'` 为核心的调用或声明。
- **L1384 EN**: Declares TableGen def record `err_pragma_attribute_invalid_argument`.
  **L1384 CN**: 声明 TableGen def 记录 `err_pragma_attribute_invalid_argument`。
- **L1385 EN**: Continues the surrounding expression or declaration: `"unexpected argument '%0' to '#pragma clang attribute'; "`.
  **L1385 CN**: 继续构造周围的表达式或声明：`"unexpected argument '%0' to '#pragma clang attribute'; "`。
- **L1386 EN**: Adds a standalone statement or declaration: `"expected 'push' or 'pop'">;`.
  **L1386 CN**: 添加一条独立语句或声明：`"expected 'push' or 'pop'">;`。
- **L1387 EN**: Declares TableGen def record `err_pragma_attribute_expected_attribute`.
  **L1387 CN**: 声明 TableGen def 记录 `err_pragma_attribute_expected_attribute`。
- **L1388 EN**: Executes a call or declaration centered on `'`.
  **L1388 CN**: 执行以 `'` 为核心的调用或声明。
- **L1389 EN**: Declares TableGen def record `err_pragma_attribute_expected_attribute_name`.
  **L1389 CN**: 声明 TableGen def 记录 `err_pragma_attribute_expected_attribute_name`。
- **L1390 EN**: Adds a standalone statement or declaration: `"expected identifier that represents an attribute name">;`.
  **L1390 CN**: 添加一条独立语句或声明：`"expected identifier that represents an attribute name">;`。
- **L1391 EN**: Declares TableGen def record `err_pragma_attribute_extra_tokens_after_attribute`.
  **L1391 CN**: 声明 TableGen def 记录 `err_pragma_attribute_extra_tokens_after_attribute`。
- **L1392 EN**: Adds a standalone statement or declaration: `"extra tokens after attribute in a '#pragma clang attribute push'">;`.
  **L1392 CN**: 添加一条独立语句或声明：`"extra tokens after attribute in a '#pragma clang attribute push'">;`。

### Lines 1393-1416

````tablegen
def err_pragma_attribute_unsupported_attribute : Error<
  "attribute %0 is not supported by '#pragma clang attribute'">;
def err_pragma_attribute_expected_attribute_syntax : Error<
  "expected an attribute that is specified using the GNU, C++11 or '__declspec'"
  " syntax">;
def note_pragma_attribute_use_attribute_kw : Note<"use the GNU '__attribute__' "
  "syntax">;
def err_pragma_attribute_invalid_subject_set_specifier : Error<
  "expected attribute subject set specifier 'apply_to'">;
def err_pragma_attribute_expected_subject_identifier : Error<
  "expected an identifier that corresponds to an attribute subject rule">;
def err_pragma_attribute_unknown_subject_rule : Error<
  "unknown attribute subject rule '%0'">;
def err_pragma_attribute_expected_subject_sub_identifier : Error<
  "expected an identifier that corresponds to an attribute subject matcher "
  "sub-rule; '%0' matcher %select{does not support sub-rules|supports the "
  "following sub-rules: %2|}1">;
def err_pragma_attribute_unknown_subject_sub_rule : Error<
  "%select{invalid use of|unknown}2 attribute subject matcher sub-rule '%0'; "
  "'%1' matcher %select{does not support sub-rules|supports the following "
  "sub-rules: %3}2">;
def err_pragma_attribute_duplicate_subject : Error<
  "duplicate attribute subject matcher '%0'">;
def err_pragma_attribute_expected_period : Error<
````
- **L1393 EN**: Declares TableGen def record `err_pragma_attribute_unsupported_attribute`.
  **L1393 CN**: 声明 TableGen def 记录 `err_pragma_attribute_unsupported_attribute`。
- **L1394 EN**: Adds a standalone statement or declaration: `"attribute %0 is not supported by '#pragma clang attribute'">;`.
  **L1394 CN**: 添加一条独立语句或声明：`"attribute %0 is not supported by '#pragma clang attribute'">;`。
- **L1395 EN**: Declares TableGen def record `err_pragma_attribute_expected_attribute_syntax`.
  **L1395 CN**: 声明 TableGen def 记录 `err_pragma_attribute_expected_attribute_syntax`。
- **L1396 EN**: Applies compiler-specific attributes that refine ABI, layout, inlining, or diagnostics: `"expected an attribute that is specified using the GNU, C++11 or '__declspec'"`.
  **L1396 CN**: 应用编译器特定属性，以细化 ABI、布局、内联或诊断行为：`"expected an attribute that is specified using the GNU, C++11 or '__declspec'"`。
- **L1397 EN**: Adds a standalone statement or declaration: `" syntax">;`.
  **L1397 CN**: 添加一条独立语句或声明：`" syntax">;`。
- **L1398 EN**: Declares TableGen def record `note_pragma_attribute_use_attribute_kw`.
  **L1398 CN**: 声明 TableGen def 记录 `note_pragma_attribute_use_attribute_kw`。
- **L1399 EN**: Adds a standalone statement or declaration: `"syntax">;`.
  **L1399 CN**: 添加一条独立语句或声明：`"syntax">;`。
- **L1400 EN**: Declares TableGen def record `err_pragma_attribute_invalid_subject_set_specifier`.
  **L1400 CN**: 声明 TableGen def 记录 `err_pragma_attribute_invalid_subject_set_specifier`。
- **L1401 EN**: Adds a standalone statement or declaration: `"expected attribute subject set specifier 'apply_to'">;`.
  **L1401 CN**: 添加一条独立语句或声明：`"expected attribute subject set specifier 'apply_to'">;`。
- **L1402 EN**: Declares TableGen def record `err_pragma_attribute_expected_subject_identifier`.
  **L1402 CN**: 声明 TableGen def 记录 `err_pragma_attribute_expected_subject_identifier`。
- **L1403 EN**: Adds a standalone statement or declaration: `"expected an identifier that corresponds to an attribute subject rule">;`.
  **L1403 CN**: 添加一条独立语句或声明：`"expected an identifier that corresponds to an attribute subject rule">;`。
- **L1404 EN**: Declares TableGen def record `err_pragma_attribute_unknown_subject_rule`.
  **L1404 CN**: 声明 TableGen def 记录 `err_pragma_attribute_unknown_subject_rule`。
- **L1405 EN**: Adds a standalone statement or declaration: `"unknown attribute subject rule '%0'">;`.
  **L1405 CN**: 添加一条独立语句或声明：`"unknown attribute subject rule '%0'">;`。
- **L1406 EN**: Declares TableGen def record `err_pragma_attribute_expected_subject_sub_identifier`.
  **L1406 CN**: 声明 TableGen def 记录 `err_pragma_attribute_expected_subject_sub_identifier`。
- **L1407 EN**: Continues the surrounding expression or declaration: `"expected an identifier that corresponds to an attribute subject matcher "`.
  **L1407 CN**: 继续构造周围的表达式或声明：`"expected an identifier that corresponds to an attribute subject matcher "`。
- **L1408 EN**: Continues the surrounding expression or declaration: `"sub-rule; '%0' matcher %select{does not support sub-rules|supports the "`.
  **L1408 CN**: 继续构造周围的表达式或声明：`"sub-rule; '%0' matcher %select{does not support sub-rules|supports the "`。
- **L1409 EN**: Adds a standalone statement or declaration: `"following sub-rules: %2|}1">;`.
  **L1409 CN**: 添加一条独立语句或声明：`"following sub-rules: %2|}1">;`。
- **L1410 EN**: Declares TableGen def record `err_pragma_attribute_unknown_subject_sub_rule`.
  **L1410 CN**: 声明 TableGen def 记录 `err_pragma_attribute_unknown_subject_sub_rule`。
- **L1411 EN**: Continues the surrounding expression or declaration: `"%select{invalid use of|unknown}2 attribute subject matcher sub-rule '%0'; "`.
  **L1411 CN**: 继续构造周围的表达式或声明：`"%select{invalid use of|unknown}2 attribute subject matcher sub-rule '%0'; "`。
- **L1412 EN**: Continues the surrounding expression or declaration: `"'%1' matcher %select{does not support sub-rules|supports the following "`.
  **L1412 CN**: 继续构造周围的表达式或声明：`"'%1' matcher %select{does not support sub-rules|supports the following "`。
- **L1413 EN**: Adds a standalone statement or declaration: `"sub-rules: %3}2">;`.
  **L1413 CN**: 添加一条独立语句或声明：`"sub-rules: %3}2">;`。
- **L1414 EN**: Declares TableGen def record `err_pragma_attribute_duplicate_subject`.
  **L1414 CN**: 声明 TableGen def 记录 `err_pragma_attribute_duplicate_subject`。
- **L1415 EN**: Adds a standalone statement or declaration: `"duplicate attribute subject matcher '%0'">;`.
  **L1415 CN**: 添加一条独立语句或声明：`"duplicate attribute subject matcher '%0'">;`。
- **L1416 EN**: Declares TableGen def record `err_pragma_attribute_expected_period`.
  **L1416 CN**: 声明 TableGen def 记录 `err_pragma_attribute_expected_period`。

### Lines 1417-1440

````tablegen
  "expected '.' after pragma attribute namespace %0">;
def err_pragma_attribute_namespace_on_attribute : Error<
  "namespace can only apply to 'push' or 'pop' directives">;
def note_pragma_attribute_namespace_on_attribute : Note<
  "omit the namespace to add attributes to the most-recently"
  " pushed attribute group">;
def warn_no_support_for_eval_method_source_on_m32 : Warning<
  "setting the floating point evaluation method to `source` on a target "
  "without SSE is not supported">, InGroup<Pragmas>;
// - #pragma __debug
def warn_pragma_debug_dependent_argument : Warning<
  "%select{value|type}0-dependent expression passed as an argument to debug "
  "command">, InGroup<IgnoredPragmas>;

// OpenCL EXTENSION pragma (OpenCL 1.1 [9.1])
def warn_pragma_expected_colon : Warning<
  "missing ':' after %0 - ignoring">, InGroup<IgnoredPragmas>;
def warn_pragma_expected_predicate : Warning<
  "expected %select{'enable', 'disable', 'begin' or 'end'|'disable'}0 - ignoring">, InGroup<IgnoredPragmas>;
def warn_pragma_unknown_extension : Warning<
  "OpenCL extension %0 unknown or does not require pragma - ignoring">, InGroup<IgnoredPragmas>;
def warn_pragma_unsupported_extension : Warning<
  "unsupported OpenCL extension %0 - ignoring">, InGroup<IgnoredPragmas>;
def warn_pragma_extension_is_core : Warning<
````
- **L1417 EN**: Adds a standalone statement or declaration: `"expected '.' after pragma attribute namespace %0">;`.
  **L1417 CN**: 添加一条独立语句或声明：`"expected '.' after pragma attribute namespace %0">;`。
- **L1418 EN**: Declares TableGen def record `err_pragma_attribute_namespace_on_attribute`.
  **L1418 CN**: 声明 TableGen def 记录 `err_pragma_attribute_namespace_on_attribute`。
- **L1419 EN**: Adds a standalone statement or declaration: `"namespace can only apply to 'push' or 'pop' directives">;`.
  **L1419 CN**: 添加一条独立语句或声明：`"namespace can only apply to 'push' or 'pop' directives">;`。
- **L1420 EN**: Declares TableGen def record `note_pragma_attribute_namespace_on_attribute`.
  **L1420 CN**: 声明 TableGen def 记录 `note_pragma_attribute_namespace_on_attribute`。
- **L1421 EN**: Continues the surrounding expression or declaration: `"omit the namespace to add attributes to the most-recently"`.
  **L1421 CN**: 继续构造周围的表达式或声明：`"omit the namespace to add attributes to the most-recently"`。
- **L1422 EN**: Adds a standalone statement or declaration: `" pushed attribute group">;`.
  **L1422 CN**: 添加一条独立语句或声明：`" pushed attribute group">;`。
- **L1423 EN**: Declares TableGen def record `warn_no_support_for_eval_method_source_on_m32`.
  **L1423 CN**: 声明 TableGen def 记录 `warn_no_support_for_eval_method_source_on_m32`。
- **L1424 EN**: Continues the surrounding expression or declaration: `"setting the floating point evaluation method to `source` on a target "`.
  **L1424 CN**: 继续构造周围的表达式或声明：`"setting the floating point evaluation method to `source` on a target "`。
- **L1425 EN**: Adds a standalone statement or declaration: `"without SSE is not supported">, InGroup<Pragmas>;`.
  **L1425 CN**: 添加一条独立语句或声明：`"without SSE is not supported">, InGroup<Pragmas>;`。
- **L1426 EN**: Comment explains nearby logic, constraints, or intent: `#pragma __debug`.
  **L1426 CN**: 注释解释附近代码的逻辑、约束或设计意图：`#pragma __debug`。
- **L1427 EN**: Declares TableGen def record `warn_pragma_debug_dependent_argument`.
  **L1427 CN**: 声明 TableGen def 记录 `warn_pragma_debug_dependent_argument`。
- **L1428 EN**: Continues the surrounding expression or declaration: `"%select{value|type}0-dependent expression passed as an argument to debug "`.
  **L1428 CN**: 继续构造周围的表达式或声明：`"%select{value|type}0-dependent expression passed as an argument to debug "`。
- **L1429 EN**: Adds a standalone statement or declaration: `"command">, InGroup<IgnoredPragmas>;`.
  **L1429 CN**: 添加一条独立语句或声明：`"command">, InGroup<IgnoredPragmas>;`。
- **L1430 EN**: Blank line separating nearby declarations or logic blocks.
  **L1430 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1431 EN**: Comment explains nearby logic, constraints, or intent: `OpenCL EXTENSION pragma (OpenCL 1.1 [9.1])`.
  **L1431 CN**: 注释解释附近代码的逻辑、约束或设计意图：`OpenCL EXTENSION pragma (OpenCL 1.1 [9.1])`。
- **L1432 EN**: Declares TableGen def record `warn_pragma_expected_colon`.
  **L1432 CN**: 声明 TableGen def 记录 `warn_pragma_expected_colon`。
- **L1433 EN**: Adds a standalone statement or declaration: `"missing ':' after %0 - ignoring">, InGroup<IgnoredPragmas>;`.
  **L1433 CN**: 添加一条独立语句或声明：`"missing ':' after %0 - ignoring">, InGroup<IgnoredPragmas>;`。
- **L1434 EN**: Declares TableGen def record `warn_pragma_expected_predicate`.
  **L1434 CN**: 声明 TableGen def 记录 `warn_pragma_expected_predicate`。
- **L1435 EN**: Adds a standalone statement or declaration: `"expected %select{'enable', 'disable', 'begin' or 'end'|'disable'}0 - ignoring">, InGroup<IgnoredPragmas>;`.
  **L1435 CN**: 添加一条独立语句或声明：`"expected %select{'enable', 'disable', 'begin' or 'end'|'disable'}0 - ignoring">, InGroup<IgnoredPragmas>;`。
- **L1436 EN**: Declares TableGen def record `warn_pragma_unknown_extension`.
  **L1436 CN**: 声明 TableGen def 记录 `warn_pragma_unknown_extension`。
- **L1437 EN**: Adds a standalone statement or declaration: `"OpenCL extension %0 unknown or does not require pragma - ignoring">, InGroup<IgnoredPragmas>;`.
  **L1437 CN**: 添加一条独立语句或声明：`"OpenCL extension %0 unknown or does not require pragma - ignoring">, InGroup<IgnoredPragmas>;`。
- **L1438 EN**: Declares TableGen def record `warn_pragma_unsupported_extension`.
  **L1438 CN**: 声明 TableGen def 记录 `warn_pragma_unsupported_extension`。
- **L1439 EN**: Adds a standalone statement or declaration: `"unsupported OpenCL extension %0 - ignoring">, InGroup<IgnoredPragmas>;`.
  **L1439 CN**: 添加一条独立语句或声明：`"unsupported OpenCL extension %0 - ignoring">, InGroup<IgnoredPragmas>;`。
- **L1440 EN**: Declares TableGen def record `warn_pragma_extension_is_core`.
  **L1440 CN**: 声明 TableGen def 记录 `warn_pragma_extension_is_core`。

### Lines 1441-1464

````tablegen
  "OpenCL extension %0 is core feature or supported optional core feature - ignoring">,
  InGroup<OpenCLCoreFeaturesDiagGroup>, DefaultIgnore;
def err_modifier_expected_colon : Error<"missing ':' after %0 modifier">;

// OpenCL errors.
def err_opencl_taking_function_address_parser : Error<
  "taking address of function is not allowed">;

// C++ for OpenCL.
def err_openclcxx_virtual_function : Error<
  "virtual functions are not supported in C++ for OpenCL">;

// OpenACC Support.
def warn_pragma_acc_ignored
    : Warning<"unexpected '#pragma acc ...' in program">,
      InGroup<SourceUsesOpenACC>,
      DefaultIgnore;
def err_acc_unexpected_directive
    : Error<"unexpected OpenACC directive %select{|'#pragma acc %1'}0">;
def err_acc_invalid_directive
    : Error<"invalid OpenACC directive %select{%1|'%1 %2'}0">;
def err_acc_invalid_clause : Error<"invalid OpenACC clause %0">;
def warn_acc_unsupported_extension_clause
    : Warning<"unsupported OpenACC extension clause %0">,
````
- **L1441 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"OpenCL extension %0 is core feature or supported optional core feature - ignoring">,`.
  **L1441 CN**: 继续一个多行参数列表、初始化器或聚合项：`"OpenCL extension %0 is core feature or supported optional core feature - ignoring">,`。
- **L1442 EN**: Adds a standalone statement or declaration: `InGroup<OpenCLCoreFeaturesDiagGroup>, DefaultIgnore;`.
  **L1442 CN**: 添加一条独立语句或声明：`InGroup<OpenCLCoreFeaturesDiagGroup>, DefaultIgnore;`。
- **L1443 EN**: Declares TableGen def record `err_modifier_expected_colon`.
  **L1443 CN**: 声明 TableGen def 记录 `err_modifier_expected_colon`。
- **L1444 EN**: Blank line separating nearby declarations or logic blocks.
  **L1444 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1445 EN**: Comment explains nearby logic, constraints, or intent: `OpenCL errors.`.
  **L1445 CN**: 注释解释附近代码的逻辑、约束或设计意图：`OpenCL errors.`。
- **L1446 EN**: Declares TableGen def record `err_opencl_taking_function_address_parser`.
  **L1446 CN**: 声明 TableGen def 记录 `err_opencl_taking_function_address_parser`。
- **L1447 EN**: Adds a standalone statement or declaration: `"taking address of function is not allowed">;`.
  **L1447 CN**: 添加一条独立语句或声明：`"taking address of function is not allowed">;`。
- **L1448 EN**: Blank line separating nearby declarations or logic blocks.
  **L1448 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1449 EN**: Comment explains nearby logic, constraints, or intent: `C++ for OpenCL.`.
  **L1449 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C++ for OpenCL.`。
- **L1450 EN**: Declares TableGen def record `err_openclcxx_virtual_function`.
  **L1450 CN**: 声明 TableGen def 记录 `err_openclcxx_virtual_function`。
- **L1451 EN**: Adds a standalone statement or declaration: `"virtual functions are not supported in C++ for OpenCL">;`.
  **L1451 CN**: 添加一条独立语句或声明：`"virtual functions are not supported in C++ for OpenCL">;`。
- **L1452 EN**: Blank line separating nearby declarations or logic blocks.
  **L1452 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1453 EN**: Comment explains nearby logic, constraints, or intent: `OpenACC Support.`.
  **L1453 CN**: 注释解释附近代码的逻辑、约束或设计意图：`OpenACC Support.`。
- **L1454 EN**: Declares TableGen def record `warn_pragma_acc_ignored`.
  **L1454 CN**: 声明 TableGen def 记录 `warn_pragma_acc_ignored`。
- **L1455 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Warning<"unexpected '#pragma acc ...' in program">,`.
  **L1455 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Warning<"unexpected '#pragma acc ...' in program">,`。
- **L1456 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `InGroup<SourceUsesOpenACC>,`.
  **L1456 CN**: 继续一个多行参数列表、初始化器或聚合项：`InGroup<SourceUsesOpenACC>,`。
- **L1457 EN**: Adds a standalone statement or declaration: `DefaultIgnore;`.
  **L1457 CN**: 添加一条独立语句或声明：`DefaultIgnore;`。
- **L1458 EN**: Declares TableGen def record `err_acc_unexpected_directive`.
  **L1458 CN**: 声明 TableGen def 记录 `err_acc_unexpected_directive`。
- **L1459 EN**: Adds a standalone statement or declaration: `: Error<"unexpected OpenACC directive %select{|'#pragma acc %1'}0">;`.
  **L1459 CN**: 添加一条独立语句或声明：`: Error<"unexpected OpenACC directive %select{|'#pragma acc %1'}0">;`。
- **L1460 EN**: Declares TableGen def record `err_acc_invalid_directive`.
  **L1460 CN**: 声明 TableGen def 记录 `err_acc_invalid_directive`。
- **L1461 EN**: Adds a standalone statement or declaration: `: Error<"invalid OpenACC directive %select{%1|'%1 %2'}0">;`.
  **L1461 CN**: 添加一条独立语句或声明：`: Error<"invalid OpenACC directive %select{%1|'%1 %2'}0">;`。
- **L1462 EN**: Declares TableGen def record `err_acc_invalid_clause`.
  **L1462 CN**: 声明 TableGen def 记录 `err_acc_invalid_clause`。
- **L1463 EN**: Declares TableGen def record `warn_acc_unsupported_extension_clause`.
  **L1463 CN**: 声明 TableGen def 记录 `warn_acc_unsupported_extension_clause`。
- **L1464 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Warning<"unsupported OpenACC extension clause %0">,`.
  **L1464 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Warning<"unsupported OpenACC extension clause %0">,`。

### Lines 1465-1488

````tablegen
      InGroup<DiagGroup<"unknown-acc-extension-clause">>;
def err_acc_missing_directive : Error<"expected OpenACC directive">;
def err_acc_invalid_open_paren
    : Error<"expected clause-list or newline in OpenACC directive">;
def err_acc_invalid_default_clause_kind
    : Error<"invalid value for 'default' clause; expected 'present' or 'none'">;
def err_acc_invalid_tag_kind
    : Error<"invalid tag %0 on '%1' %select{directive|clause}2">;
def err_acc_expected_reduction_operator
    : Error<"missing reduction operator, expected '+', '*', 'max', 'min', '&', "
            "'|', '^', '&&', or '||', follwed by a ':'">;
def err_acc_invalid_reduction_operator
    : Error<"invalid reduction operator,  expected '+', '*', 'max', 'min', "
            "'&', '|', '^', '&&', or '||'">;
def err_acc_incorrect_bind_arg
    : Error<"expected identifier or string literal in OpenACC 'bind' clause">;
def err_acc_modifier
    : Error<"%enum_select<ACCModifier>{%Unknown{unknown}|%Duplicate{duplicate}}"
            "0 modifier %1 in OpenACC modifier-list on '%2' clause">;

// OpenMP support.
def warn_pragma_omp_ignored : Warning<
  "unexpected '#pragma omp ...' in program">, InGroup<SourceUsesOpenMP>, DefaultIgnore;
def warn_omp_extra_tokens_at_eol : Warning<
````
- **L1465 EN**: Adds a standalone statement or declaration: `InGroup<DiagGroup<"unknown-acc-extension-clause">>;`.
  **L1465 CN**: 添加一条独立语句或声明：`InGroup<DiagGroup<"unknown-acc-extension-clause">>;`。
- **L1466 EN**: Declares TableGen def record `err_acc_missing_directive`.
  **L1466 CN**: 声明 TableGen def 记录 `err_acc_missing_directive`。
- **L1467 EN**: Declares TableGen def record `err_acc_invalid_open_paren`.
  **L1467 CN**: 声明 TableGen def 记录 `err_acc_invalid_open_paren`。
- **L1468 EN**: Adds a standalone statement or declaration: `: Error<"expected clause-list or newline in OpenACC directive">;`.
  **L1468 CN**: 添加一条独立语句或声明：`: Error<"expected clause-list or newline in OpenACC directive">;`。
- **L1469 EN**: Declares TableGen def record `err_acc_invalid_default_clause_kind`.
  **L1469 CN**: 声明 TableGen def 记录 `err_acc_invalid_default_clause_kind`。
- **L1470 EN**: Adds a standalone statement or declaration: `: Error<"invalid value for 'default' clause; expected 'present' or 'none'">;`.
  **L1470 CN**: 添加一条独立语句或声明：`: Error<"invalid value for 'default' clause; expected 'present' or 'none'">;`。
- **L1471 EN**: Declares TableGen def record `err_acc_invalid_tag_kind`.
  **L1471 CN**: 声明 TableGen def 记录 `err_acc_invalid_tag_kind`。
- **L1472 EN**: Adds a standalone statement or declaration: `: Error<"invalid tag %0 on '%1' %select{directive|clause}2">;`.
  **L1472 CN**: 添加一条独立语句或声明：`: Error<"invalid tag %0 on '%1' %select{directive|clause}2">;`。
- **L1473 EN**: Declares TableGen def record `err_acc_expected_reduction_operator`.
  **L1473 CN**: 声明 TableGen def 记录 `err_acc_expected_reduction_operator`。
- **L1474 EN**: Continues the surrounding expression or declaration: `: Error<"missing reduction operator, expected '+', '*', 'max', 'min', '&', "`.
  **L1474 CN**: 继续构造周围的表达式或声明：`: Error<"missing reduction operator, expected '+', '*', 'max', 'min', '&', "`。
- **L1475 EN**: Adds a standalone statement or declaration: `"'|', '^', '&&', or '||', follwed by a ':'">;`.
  **L1475 CN**: 添加一条独立语句或声明：`"'|', '^', '&&', or '||', follwed by a ':'">;`。
- **L1476 EN**: Declares TableGen def record `err_acc_invalid_reduction_operator`.
  **L1476 CN**: 声明 TableGen def 记录 `err_acc_invalid_reduction_operator`。
- **L1477 EN**: Continues the surrounding expression or declaration: `: Error<"invalid reduction operator,  expected '+', '*', 'max', 'min', "`.
  **L1477 CN**: 继续构造周围的表达式或声明：`: Error<"invalid reduction operator,  expected '+', '*', 'max', 'min', "`。
- **L1478 EN**: Adds a standalone statement or declaration: `"'&', '|', '^', '&&', or '||'">;`.
  **L1478 CN**: 添加一条独立语句或声明：`"'&', '|', '^', '&&', or '||'">;`。
- **L1479 EN**: Declares TableGen def record `err_acc_incorrect_bind_arg`.
  **L1479 CN**: 声明 TableGen def 记录 `err_acc_incorrect_bind_arg`。
- **L1480 EN**: Adds a standalone statement or declaration: `: Error<"expected identifier or string literal in OpenACC 'bind' clause">;`.
  **L1480 CN**: 添加一条独立语句或声明：`: Error<"expected identifier or string literal in OpenACC 'bind' clause">;`。
- **L1481 EN**: Declares TableGen def record `err_acc_modifier`.
  **L1481 CN**: 声明 TableGen def 记录 `err_acc_modifier`。
- **L1482 EN**: Continues the surrounding expression or declaration: `: Error<"%enum_select<ACCModifier>{%Unknown{unknown}|%Duplicate{duplicate}}"`.
  **L1482 CN**: 继续构造周围的表达式或声明：`: Error<"%enum_select<ACCModifier>{%Unknown{unknown}|%Duplicate{duplicate}}"`。
- **L1483 EN**: Adds a standalone statement or declaration: `"0 modifier %1 in OpenACC modifier-list on '%2' clause">;`.
  **L1483 CN**: 添加一条独立语句或声明：`"0 modifier %1 in OpenACC modifier-list on '%2' clause">;`。
- **L1484 EN**: Blank line separating nearby declarations or logic blocks.
  **L1484 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1485 EN**: Comment explains nearby logic, constraints, or intent: `OpenMP support.`.
  **L1485 CN**: 注释解释附近代码的逻辑、约束或设计意图：`OpenMP support.`。
- **L1486 EN**: Declares TableGen def record `warn_pragma_omp_ignored`.
  **L1486 CN**: 声明 TableGen def 记录 `warn_pragma_omp_ignored`。
- **L1487 EN**: Adds a standalone statement or declaration: `"unexpected '#pragma omp ...' in program">, InGroup<SourceUsesOpenMP>, DefaultIgnore;`.
  **L1487 CN**: 添加一条独立语句或声明：`"unexpected '#pragma omp ...' in program">, InGroup<SourceUsesOpenMP>, DefaultIgnore;`。
- **L1488 EN**: Declares TableGen def record `warn_omp_extra_tokens_at_eol`.
  **L1488 CN**: 声明 TableGen def 记录 `warn_omp_extra_tokens_at_eol`。

### Lines 1489-1512

````tablegen
  "extra tokens at the end of '#pragma omp %0' are ignored">,
  InGroup<ExtraTokens>;
def err_omp_multiple_step_or_linear_modifier : Error<
  "multiple %select{'step size'|'linear modifier'}0 found in linear clause">;
def err_omp_deprecate_old_syntax: Error<
  "old syntax '%0' on '%1' clause was deprecated, use new syntax '%2'">;
def err_omp_allocator_comma_separator :
  Error<"',' not allowed as separator in 'uses_allocators' clause, use ';' instead">;
def warn_omp_future_directive_spelling: Warning<
  "directive spelling '%0' is introduced in a later OpenMP version">,
  InGroup<OpenMPFuture>;
def warn_pragma_expected_colon_r_paren : Warning<
  "missing ':' or ')' after %0 - ignoring">, InGroup<IgnoredPragmas>;
def err_omp_unknown_directive : Error<
  "expected an OpenMP directive">;
def err_omp_unexpected_directive : Error<
  "unexpected OpenMP directive %select{|'#pragma omp %1'}0">;
def err_omp_expected_punc : Error<
  "expected ',' or ')' in '%0' %select{clause|directive}1">;
def warn_clause_expected_string: Warning<
  "expected string %select{|literal }1in 'clause %0' - ignoring">, InGroup<IgnoredPragmas>;
def err_omp_unexpected_clause : Error<
  "unexpected OpenMP clause '%0' in directive '#pragma omp %1'">;
def err_omp_unexpected_clause_extension_only : Error<
````
- **L1489 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"extra tokens at the end of '#pragma omp %0' are ignored">,`.
  **L1489 CN**: 继续一个多行参数列表、初始化器或聚合项：`"extra tokens at the end of '#pragma omp %0' are ignored">,`。
- **L1490 EN**: Adds a standalone statement or declaration: `InGroup<ExtraTokens>;`.
  **L1490 CN**: 添加一条独立语句或声明：`InGroup<ExtraTokens>;`。
- **L1491 EN**: Declares TableGen def record `err_omp_multiple_step_or_linear_modifier`.
  **L1491 CN**: 声明 TableGen def 记录 `err_omp_multiple_step_or_linear_modifier`。
- **L1492 EN**: Adds a standalone statement or declaration: `"multiple %select{'step size'|'linear modifier'}0 found in linear clause">;`.
  **L1492 CN**: 添加一条独立语句或声明：`"multiple %select{'step size'|'linear modifier'}0 found in linear clause">;`。
- **L1493 EN**: Declares TableGen def record `err_omp_deprecate_old_syntax`.
  **L1493 CN**: 声明 TableGen def 记录 `err_omp_deprecate_old_syntax`。
- **L1494 EN**: Adds a standalone statement or declaration: `"old syntax '%0' on '%1' clause was deprecated, use new syntax '%2'">;`.
  **L1494 CN**: 添加一条独立语句或声明：`"old syntax '%0' on '%1' clause was deprecated, use new syntax '%2'">;`。
- **L1495 EN**: Declares TableGen def record `err_omp_allocator_comma_separator`.
  **L1495 CN**: 声明 TableGen def 记录 `err_omp_allocator_comma_separator`。
- **L1496 EN**: Adds a standalone statement or declaration: `Error<"',' not allowed as separator in 'uses_allocators' clause, use ';' instead">;`.
  **L1496 CN**: 添加一条独立语句或声明：`Error<"',' not allowed as separator in 'uses_allocators' clause, use ';' instead">;`。
- **L1497 EN**: Declares TableGen def record `warn_omp_future_directive_spelling`.
  **L1497 CN**: 声明 TableGen def 记录 `warn_omp_future_directive_spelling`。
- **L1498 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"directive spelling '%0' is introduced in a later OpenMP version">,`.
  **L1498 CN**: 继续一个多行参数列表、初始化器或聚合项：`"directive spelling '%0' is introduced in a later OpenMP version">,`。
- **L1499 EN**: Adds a standalone statement or declaration: `InGroup<OpenMPFuture>;`.
  **L1499 CN**: 添加一条独立语句或声明：`InGroup<OpenMPFuture>;`。
- **L1500 EN**: Declares TableGen def record `warn_pragma_expected_colon_r_paren`.
  **L1500 CN**: 声明 TableGen def 记录 `warn_pragma_expected_colon_r_paren`。
- **L1501 EN**: Adds a standalone statement or declaration: `"missing ':' or ')' after %0 - ignoring">, InGroup<IgnoredPragmas>;`.
  **L1501 CN**: 添加一条独立语句或声明：`"missing ':' or ')' after %0 - ignoring">, InGroup<IgnoredPragmas>;`。
- **L1502 EN**: Declares TableGen def record `err_omp_unknown_directive`.
  **L1502 CN**: 声明 TableGen def 记录 `err_omp_unknown_directive`。
- **L1503 EN**: Adds a standalone statement or declaration: `"expected an OpenMP directive">;`.
  **L1503 CN**: 添加一条独立语句或声明：`"expected an OpenMP directive">;`。
- **L1504 EN**: Declares TableGen def record `err_omp_unexpected_directive`.
  **L1504 CN**: 声明 TableGen def 记录 `err_omp_unexpected_directive`。
- **L1505 EN**: Adds a standalone statement or declaration: `"unexpected OpenMP directive %select{|'#pragma omp %1'}0">;`.
  **L1505 CN**: 添加一条独立语句或声明：`"unexpected OpenMP directive %select{|'#pragma omp %1'}0">;`。
- **L1506 EN**: Declares TableGen def record `err_omp_expected_punc`.
  **L1506 CN**: 声明 TableGen def 记录 `err_omp_expected_punc`。
- **L1507 EN**: Adds a standalone statement or declaration: `"expected ',' or ')' in '%0' %select{clause|directive}1">;`.
  **L1507 CN**: 添加一条独立语句或声明：`"expected ',' or ')' in '%0' %select{clause|directive}1">;`。
- **L1508 EN**: Declares TableGen def record `warn_clause_expected_string`.
  **L1508 CN**: 声明 TableGen def 记录 `warn_clause_expected_string`。
- **L1509 EN**: Adds a standalone statement or declaration: `"expected string %select{|literal }1in 'clause %0' - ignoring">, InGroup<IgnoredPragmas>;`.
  **L1509 CN**: 添加一条独立语句或声明：`"expected string %select{|literal }1in 'clause %0' - ignoring">, InGroup<IgnoredPragmas>;`。
- **L1510 EN**: Declares TableGen def record `err_omp_unexpected_clause`.
  **L1510 CN**: 声明 TableGen def 记录 `err_omp_unexpected_clause`。
- **L1511 EN**: Adds a standalone statement or declaration: `"unexpected OpenMP clause '%0' in directive '#pragma omp %1'">;`.
  **L1511 CN**: 添加一条独立语句或声明：`"unexpected OpenMP clause '%0' in directive '#pragma omp %1'">;`。
- **L1512 EN**: Declares TableGen def record `err_omp_unexpected_clause_extension_only`.
  **L1512 CN**: 声明 TableGen def 记录 `err_omp_unexpected_clause_extension_only`。

### Lines 1513-1536

````tablegen
  "OpenMP clause '%0' is only available as extension, use '-fopenmp-extensions'">;
def err_omp_immediate_directive : Error<
  "'#pragma omp %0' %select{|with '%2' clause }1cannot be an immediate substatement">;
def err_omp_expected_identifier_for_critical : Error<
  "expected identifier specifying the name of the 'omp critical' directive">;
def err_omp_expected_reduction_identifier : Error<
  "expected identifier or one of the following operators: '+', '-', '*', '&', '|', '^', '&&', or '||'">;
def err_omp_expected_equal_in_iterator : Error<
  "expected '=' in iterator specifier">;
def err_omp_expected_punc_after_iterator : Error<
  "expected ',' or ')' after iterator specifier">;
def err_omp_decl_in_declare_simd_variant : Error<
  "function declaration is expected after 'declare %select{simd|variant}0' directive">;
def err_omp_sink_and_source_iteration_not_allowd: Error<" '%0 %select{sink:|source:}1' must be with '%select{omp_cur_iteration - 1|omp_cur_iteration}1'">;
def err_omp_unknown_map_type : Error<
  "incorrect map type, expected one of 'to', 'from', 'tofrom', 'alloc', 'release', or 'delete'">;
def err_omp_more_one_map_type : Error<"map type is already specified">;
def note_previous_map_type_specified_here
    : Note<"map type '%0' is previous specified here">;
def err_omp_unknown_map_type_modifier : Error<
  "incorrect map type modifier, expected one of: 'always', 'close', 'mapper'"
  "%select{|, 'present'|, 'present', 'iterator'}0%select{|, 'ompx_hold'}1%select{|, 'self'}2">;
def err_omp_map_type_missing : Error<
  "missing map type">;
````
- **L1513 EN**: Adds a standalone statement or declaration: `"OpenMP clause '%0' is only available as extension, use '-fopenmp-extensions'">;`.
  **L1513 CN**: 添加一条独立语句或声明：`"OpenMP clause '%0' is only available as extension, use '-fopenmp-extensions'">;`。
- **L1514 EN**: Declares TableGen def record `err_omp_immediate_directive`.
  **L1514 CN**: 声明 TableGen def 记录 `err_omp_immediate_directive`。
- **L1515 EN**: Adds a standalone statement or declaration: `"'#pragma omp %0' %select{|with '%2' clause }1cannot be an immediate substatement">;`.
  **L1515 CN**: 添加一条独立语句或声明：`"'#pragma omp %0' %select{|with '%2' clause }1cannot be an immediate substatement">;`。
- **L1516 EN**: Declares TableGen def record `err_omp_expected_identifier_for_critical`.
  **L1516 CN**: 声明 TableGen def 记录 `err_omp_expected_identifier_for_critical`。
- **L1517 EN**: Adds a standalone statement or declaration: `"expected identifier specifying the name of the 'omp critical' directive">;`.
  **L1517 CN**: 添加一条独立语句或声明：`"expected identifier specifying the name of the 'omp critical' directive">;`。
- **L1518 EN**: Declares TableGen def record `err_omp_expected_reduction_identifier`.
  **L1518 CN**: 声明 TableGen def 记录 `err_omp_expected_reduction_identifier`。
- **L1519 EN**: Adds a standalone statement or declaration: `"expected identifier or one of the following operators: '+', '-', '*', '&', '|', '^', '&&', or '||'">;`.
  **L1519 CN**: 添加一条独立语句或声明：`"expected identifier or one of the following operators: '+', '-', '*', '&', '|', '^', '&&', or '||'">;`。
- **L1520 EN**: Declares TableGen def record `err_omp_expected_equal_in_iterator`.
  **L1520 CN**: 声明 TableGen def 记录 `err_omp_expected_equal_in_iterator`。
- **L1521 EN**: Adds a standalone statement or declaration: `"expected '=' in iterator specifier">;`.
  **L1521 CN**: 添加一条独立语句或声明：`"expected '=' in iterator specifier">;`。
- **L1522 EN**: Declares TableGen def record `err_omp_expected_punc_after_iterator`.
  **L1522 CN**: 声明 TableGen def 记录 `err_omp_expected_punc_after_iterator`。
- **L1523 EN**: Adds a standalone statement or declaration: `"expected ',' or ')' after iterator specifier">;`.
  **L1523 CN**: 添加一条独立语句或声明：`"expected ',' or ')' after iterator specifier">;`。
- **L1524 EN**: Declares TableGen def record `err_omp_decl_in_declare_simd_variant`.
  **L1524 CN**: 声明 TableGen def 记录 `err_omp_decl_in_declare_simd_variant`。
- **L1525 EN**: Adds a standalone statement or declaration: `"function declaration is expected after 'declare %select{simd|variant}0' directive">;`.
  **L1525 CN**: 添加一条独立语句或声明：`"function declaration is expected after 'declare %select{simd|variant}0' directive">;`。
- **L1526 EN**: Declares TableGen def record `err_omp_sink_and_source_iteration_not_allowd`.
  **L1526 CN**: 声明 TableGen def 记录 `err_omp_sink_and_source_iteration_not_allowd`。
- **L1527 EN**: Declares TableGen def record `err_omp_unknown_map_type`.
  **L1527 CN**: 声明 TableGen def 记录 `err_omp_unknown_map_type`。
- **L1528 EN**: Adds a standalone statement or declaration: `"incorrect map type, expected one of 'to', 'from', 'tofrom', 'alloc', 'release', or 'delete'">;`.
  **L1528 CN**: 添加一条独立语句或声明：`"incorrect map type, expected one of 'to', 'from', 'tofrom', 'alloc', 'release', or 'delete'">;`。
- **L1529 EN**: Declares TableGen def record `err_omp_more_one_map_type`.
  **L1529 CN**: 声明 TableGen def 记录 `err_omp_more_one_map_type`。
- **L1530 EN**: Declares TableGen def record `note_previous_map_type_specified_here`.
  **L1530 CN**: 声明 TableGen def 记录 `note_previous_map_type_specified_here`。
- **L1531 EN**: Adds a standalone statement or declaration: `: Note<"map type '%0' is previous specified here">;`.
  **L1531 CN**: 添加一条独立语句或声明：`: Note<"map type '%0' is previous specified here">;`。
- **L1532 EN**: Declares TableGen def record `err_omp_unknown_map_type_modifier`.
  **L1532 CN**: 声明 TableGen def 记录 `err_omp_unknown_map_type_modifier`。
- **L1533 EN**: Continues the surrounding expression or declaration: `"incorrect map type modifier, expected one of: 'always', 'close', 'mapper'"`.
  **L1533 CN**: 继续构造周围的表达式或声明：`"incorrect map type modifier, expected one of: 'always', 'close', 'mapper'"`。
- **L1534 EN**: Adds a standalone statement or declaration: `"%select{|, 'present'|, 'present', 'iterator'}0%select{|, 'ompx_hold'}1%select{|, 'self'}2">;`.
  **L1534 CN**: 添加一条独立语句或声明：`"%select{|, 'present'|, 'present', 'iterator'}0%select{|, 'ompx_hold'}1%select{|, 'self'}2">;`。
- **L1535 EN**: Declares TableGen def record `err_omp_map_type_missing`.
  **L1535 CN**: 声明 TableGen def 记录 `err_omp_map_type_missing`。
- **L1536 EN**: Adds a standalone statement or declaration: `"missing map type">;`.
  **L1536 CN**: 添加一条独立语句或声明：`"missing map type">;`。

### Lines 1537-1560

````tablegen
def err_omp_map_type_modifier_missing : Error<
  "missing map type modifier">;
def err_omp_map_modifier_specification_list : Error<
  "empty modifier-specification-list is not allowed">;
def err_omp_declare_simd_inbranch_notinbranch : Error<
  "unexpected '%0' clause, '%1' is specified already">;
def err_omp_expected_clause_argument
    : Error<"expected '%0' clause with an argument on '#pragma omp %1' "
            "construct">;
def err_expected_end_declare_target_or_variant : Error<
  "expected '#pragma omp end declare %select{target|variant}0'">;
def err_expected_begin_declare_variant
    : Error<"'#pragma omp end declare variant' with no matching '#pragma omp "
            "begin declare variant'">;
def err_expected_begin_assumes
    : Error<"'#pragma omp end assumes' with no matching '#pragma omp begin assumes'">;
def warn_omp_unknown_assumption_clause_missing_id
    : Warning<"valid %0 clauses start with %1; %select{token|tokens}2 will be ignored">,
      InGroup<OpenMPClauses>;
def warn_omp_unknown_assumption_clause_without_args
    : Warning<"%0 clause should not be followed by arguments; tokens will be ignored">,
      InGroup<OpenMPClauses>;
def note_omp_assumption_clause_continue_here
    : Note<"the ignored tokens spans until here">;
````
- **L1537 EN**: Declares TableGen def record `err_omp_map_type_modifier_missing`.
  **L1537 CN**: 声明 TableGen def 记录 `err_omp_map_type_modifier_missing`。
- **L1538 EN**: Adds a standalone statement or declaration: `"missing map type modifier">;`.
  **L1538 CN**: 添加一条独立语句或声明：`"missing map type modifier">;`。
- **L1539 EN**: Declares TableGen def record `err_omp_map_modifier_specification_list`.
  **L1539 CN**: 声明 TableGen def 记录 `err_omp_map_modifier_specification_list`。
- **L1540 EN**: Adds a standalone statement or declaration: `"empty modifier-specification-list is not allowed">;`.
  **L1540 CN**: 添加一条独立语句或声明：`"empty modifier-specification-list is not allowed">;`。
- **L1541 EN**: Declares TableGen def record `err_omp_declare_simd_inbranch_notinbranch`.
  **L1541 CN**: 声明 TableGen def 记录 `err_omp_declare_simd_inbranch_notinbranch`。
- **L1542 EN**: Adds a standalone statement or declaration: `"unexpected '%0' clause, '%1' is specified already">;`.
  **L1542 CN**: 添加一条独立语句或声明：`"unexpected '%0' clause, '%1' is specified already">;`。
- **L1543 EN**: Declares TableGen def record `err_omp_expected_clause_argument`.
  **L1543 CN**: 声明 TableGen def 记录 `err_omp_expected_clause_argument`。
- **L1544 EN**: Continues the surrounding expression or declaration: `: Error<"expected '%0' clause with an argument on '#pragma omp %1' "`.
  **L1544 CN**: 继续构造周围的表达式或声明：`: Error<"expected '%0' clause with an argument on '#pragma omp %1' "`。
- **L1545 EN**: Adds a standalone statement or declaration: `"construct">;`.
  **L1545 CN**: 添加一条独立语句或声明：`"construct">;`。
- **L1546 EN**: Declares TableGen def record `err_expected_end_declare_target_or_variant`.
  **L1546 CN**: 声明 TableGen def 记录 `err_expected_end_declare_target_or_variant`。
- **L1547 EN**: Adds a standalone statement or declaration: `"expected '#pragma omp end declare %select{target|variant}0'">;`.
  **L1547 CN**: 添加一条独立语句或声明：`"expected '#pragma omp end declare %select{target|variant}0'">;`。
- **L1548 EN**: Declares TableGen def record `err_expected_begin_declare_variant`.
  **L1548 CN**: 声明 TableGen def 记录 `err_expected_begin_declare_variant`。
- **L1549 EN**: Continues the surrounding expression or declaration: `: Error<"'#pragma omp end declare variant' with no matching '#pragma omp "`.
  **L1549 CN**: 继续构造周围的表达式或声明：`: Error<"'#pragma omp end declare variant' with no matching '#pragma omp "`。
- **L1550 EN**: Adds a standalone statement or declaration: `"begin declare variant'">;`.
  **L1550 CN**: 添加一条独立语句或声明：`"begin declare variant'">;`。
- **L1551 EN**: Declares TableGen def record `err_expected_begin_assumes`.
  **L1551 CN**: 声明 TableGen def 记录 `err_expected_begin_assumes`。
- **L1552 EN**: Adds a standalone statement or declaration: `: Error<"'#pragma omp end assumes' with no matching '#pragma omp begin assumes'">;`.
  **L1552 CN**: 添加一条独立语句或声明：`: Error<"'#pragma omp end assumes' with no matching '#pragma omp begin assumes'">;`。
- **L1553 EN**: Declares TableGen def record `warn_omp_unknown_assumption_clause_missing_id`.
  **L1553 CN**: 声明 TableGen def 记录 `warn_omp_unknown_assumption_clause_missing_id`。
- **L1554 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Warning<"valid %0 clauses start with %1; %select{token|tokens}2 will be ignored">,`.
  **L1554 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Warning<"valid %0 clauses start with %1; %select{token|tokens}2 will be ignored">,`。
- **L1555 EN**: Adds a standalone statement or declaration: `InGroup<OpenMPClauses>;`.
  **L1555 CN**: 添加一条独立语句或声明：`InGroup<OpenMPClauses>;`。
- **L1556 EN**: Declares TableGen def record `warn_omp_unknown_assumption_clause_without_args`.
  **L1556 CN**: 声明 TableGen def 记录 `warn_omp_unknown_assumption_clause_without_args`。
- **L1557 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Warning<"%0 clause should not be followed by arguments; tokens will be ignored">,`.
  **L1557 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Warning<"%0 clause should not be followed by arguments; tokens will be ignored">,`。
- **L1558 EN**: Adds a standalone statement or declaration: `InGroup<OpenMPClauses>;`.
  **L1558 CN**: 添加一条独立语句或声明：`InGroup<OpenMPClauses>;`。
- **L1559 EN**: Declares TableGen def record `note_omp_assumption_clause_continue_here`.
  **L1559 CN**: 声明 TableGen def 记录 `note_omp_assumption_clause_continue_here`。
- **L1560 EN**: Adds a standalone statement or declaration: `: Note<"the ignored tokens spans until here">;`.
  **L1560 CN**: 添加一条独立语句或声明：`: Note<"the ignored tokens spans until here">;`。

### Lines 1561-1584

````tablegen
def err_omp_declare_target_unexpected_clause: Error<
  "unexpected '%0' clause, only %select{'device_type'|'to' or 'link'|'to', 'link' or 'device_type'|'device_type', 'indirect'|'to', 'link', 'device_type' or 'indirect'}1 clauses expected">;
def err_omp_declare_target_unexpected_clause_52
    : Error<"unexpected '%0' clause, only %select{'device_type'|'enter' or "
            "'link'|'enter', 'link' or 'device_type'|'device_type', "
            "'indirect'|'enter', 'link', 'device_type' or 'indirect'|'enter', "
            "'link', 'device_type', 'indirect' or 'local'}1 clauses expected">;
def err_omp_begin_declare_target_unexpected_implicit_to_clause: Error<
  "unexpected '(', only 'to', 'link' or 'device_type' clauses expected for 'begin declare target' directive">;
def err_omp_declare_target_wrong_clause_after_implicit_to: Error<
  "unexpected clause after an implicit 'to' clause">;
def err_omp_declare_target_wrong_clause_after_implicit_enter: Error<
  "unexpected clause after an implicit 'enter' clause">;
def err_omp_declare_target_missing_required_clause
    : Error<"expected at least one %select{'to' or 'link'|'to', 'link' or "
            "'indirect'|'enter', 'link' or 'indirect'|'enter', 'link', "
            "'indirect' or 'local'}0 clause">;
def err_omp_declare_target_unexpected_to_clause: Error<
  "unexpected 'to' clause, use 'enter' instead">;
def err_omp_declare_target_unexpected_enter_clause: Error<
  "unexpected 'enter' clause, use 'to' instead">;
def err_omp_declare_target_multiple : Error<
  "%0 appears multiple times in clauses on the same declare target directive">;
def err_omp_declare_target_indirect_device_type: Error<
````
- **L1561 EN**: Declares TableGen def record `err_omp_declare_target_unexpected_clause`.
  **L1561 CN**: 声明 TableGen def 记录 `err_omp_declare_target_unexpected_clause`。
- **L1562 EN**: Adds a standalone statement or declaration: `"unexpected '%0' clause, only %select{'device_type'|'to' or 'link'|'to', 'link' or 'device_type'|'device_type', 'indirect'|'to', 'link', 'device_type' or 'indirect'}1 clauses expected">;`.
  **L1562 CN**: 添加一条独立语句或声明：`"unexpected '%0' clause, only %select{'device_type'|'to' or 'link'|'to', 'link' or 'device_type'|'device_type', 'indirect'|'to', 'link', 'device_type' or 'indirect'}1 clauses expected">;`。
- **L1563 EN**: Declares TableGen def record `err_omp_declare_target_unexpected_clause_52`.
  **L1563 CN**: 声明 TableGen def 记录 `err_omp_declare_target_unexpected_clause_52`。
- **L1564 EN**: Continues the surrounding expression or declaration: `: Error<"unexpected '%0' clause, only %select{'device_type'|'enter' or "`.
  **L1564 CN**: 继续构造周围的表达式或声明：`: Error<"unexpected '%0' clause, only %select{'device_type'|'enter' or "`。
- **L1565 EN**: Continues the surrounding expression or declaration: `"'link'|'enter', 'link' or 'device_type'|'device_type', "`.
  **L1565 CN**: 继续构造周围的表达式或声明：`"'link'|'enter', 'link' or 'device_type'|'device_type', "`。
- **L1566 EN**: Continues the surrounding expression or declaration: `"'indirect'|'enter', 'link', 'device_type' or 'indirect'|'enter', "`.
  **L1566 CN**: 继续构造周围的表达式或声明：`"'indirect'|'enter', 'link', 'device_type' or 'indirect'|'enter', "`。
- **L1567 EN**: Adds a standalone statement or declaration: `"'link', 'device_type', 'indirect' or 'local'}1 clauses expected">;`.
  **L1567 CN**: 添加一条独立语句或声明：`"'link', 'device_type', 'indirect' or 'local'}1 clauses expected">;`。
- **L1568 EN**: Declares TableGen def record `err_omp_begin_declare_target_unexpected_implicit_to_clause`.
  **L1568 CN**: 声明 TableGen def 记录 `err_omp_begin_declare_target_unexpected_implicit_to_clause`。
- **L1569 EN**: Executes a call or declaration centered on `'`.
  **L1569 CN**: 执行以 `'` 为核心的调用或声明。
- **L1570 EN**: Declares TableGen def record `err_omp_declare_target_wrong_clause_after_implicit_to`.
  **L1570 CN**: 声明 TableGen def 记录 `err_omp_declare_target_wrong_clause_after_implicit_to`。
- **L1571 EN**: Adds a standalone statement or declaration: `"unexpected clause after an implicit 'to' clause">;`.
  **L1571 CN**: 添加一条独立语句或声明：`"unexpected clause after an implicit 'to' clause">;`。
- **L1572 EN**: Declares TableGen def record `err_omp_declare_target_wrong_clause_after_implicit_enter`.
  **L1572 CN**: 声明 TableGen def 记录 `err_omp_declare_target_wrong_clause_after_implicit_enter`。
- **L1573 EN**: Adds a standalone statement or declaration: `"unexpected clause after an implicit 'enter' clause">;`.
  **L1573 CN**: 添加一条独立语句或声明：`"unexpected clause after an implicit 'enter' clause">;`。
- **L1574 EN**: Declares TableGen def record `err_omp_declare_target_missing_required_clause`.
  **L1574 CN**: 声明 TableGen def 记录 `err_omp_declare_target_missing_required_clause`。
- **L1575 EN**: Continues the surrounding expression or declaration: `: Error<"expected at least one %select{'to' or 'link'|'to', 'link' or "`.
  **L1575 CN**: 继续构造周围的表达式或声明：`: Error<"expected at least one %select{'to' or 'link'|'to', 'link' or "`。
- **L1576 EN**: Continues the surrounding expression or declaration: `"'indirect'|'enter', 'link' or 'indirect'|'enter', 'link', "`.
  **L1576 CN**: 继续构造周围的表达式或声明：`"'indirect'|'enter', 'link' or 'indirect'|'enter', 'link', "`。
- **L1577 EN**: Adds a standalone statement or declaration: `"'indirect' or 'local'}0 clause">;`.
  **L1577 CN**: 添加一条独立语句或声明：`"'indirect' or 'local'}0 clause">;`。
- **L1578 EN**: Declares TableGen def record `err_omp_declare_target_unexpected_to_clause`.
  **L1578 CN**: 声明 TableGen def 记录 `err_omp_declare_target_unexpected_to_clause`。
- **L1579 EN**: Adds a standalone statement or declaration: `"unexpected 'to' clause, use 'enter' instead">;`.
  **L1579 CN**: 添加一条独立语句或声明：`"unexpected 'to' clause, use 'enter' instead">;`。
- **L1580 EN**: Declares TableGen def record `err_omp_declare_target_unexpected_enter_clause`.
  **L1580 CN**: 声明 TableGen def 记录 `err_omp_declare_target_unexpected_enter_clause`。
- **L1581 EN**: Adds a standalone statement or declaration: `"unexpected 'enter' clause, use 'to' instead">;`.
  **L1581 CN**: 添加一条独立语句或声明：`"unexpected 'enter' clause, use 'to' instead">;`。
- **L1582 EN**: Declares TableGen def record `err_omp_declare_target_multiple`.
  **L1582 CN**: 声明 TableGen def 记录 `err_omp_declare_target_multiple`。
- **L1583 EN**: Adds a standalone statement or declaration: `"%0 appears multiple times in clauses on the same declare target directive">;`.
  **L1583 CN**: 添加一条独立语句或声明：`"%0 appears multiple times in clauses on the same declare target directive">;`。
- **L1584 EN**: Declares TableGen def record `err_omp_declare_target_indirect_device_type`.
  **L1584 CN**: 声明 TableGen def 记录 `err_omp_declare_target_indirect_device_type`。

### Lines 1585-1608

````tablegen
  "only 'device_type(any)' clause is allowed with indirect clause">;
def warn_omp_deprecated_declare_target_delimited_form :
  Warning<"the delimited form of '#pragma omp declare target' without clauses is deprecated; use '#pragma omp begin declare target' instead">,
  InGroup<Deprecated>;
def err_omp_expected_clause: Error<
  "expected at least one clause on '#pragma omp %0' directive">;
def err_omp_mapper_illegal_identifier : Error<
  "illegal OpenMP user-defined mapper identifier">;
def err_omp_mapper_expected_declarator : Error<
  "expected declarator on 'omp declare mapper' directive">;
def err_omp_unexpected_append_op : Error<
  "unexpected operation specified in 'append_args' clause, expected 'interop'">;
def err_omp_unexpected_execution_modifier : Error<
  "unexpected 'execution' modifier in non-executable context">;
def err_omp_unknown_adjust_args_op
    : Error<
          "incorrect 'adjust_args' type, expected 'need_device_ptr'%select{|, "
          "'need_device_addr',}0 or 'nothing'">;
def err_omp_declare_variant_wrong_clause : Error<
  "expected %select{'match'|'match', 'adjust_args', or 'append_args'}0 clause "
  "on 'omp declare variant' directive">;
def err_omp_non_by_ref_need_device_addr_modifier_argument
    : Error<"expected reference type argument on 'adjust_args' clause with "
            "'need_device_addr' modifier">;
````
- **L1585 EN**: Executes a call or declaration centered on `'device_type`.
  **L1585 CN**: 执行以 `'device_type` 为核心的调用或声明。
- **L1586 EN**: Declares TableGen def record `warn_omp_deprecated_declare_target_delimited_form`.
  **L1586 CN**: 声明 TableGen def 记录 `warn_omp_deprecated_declare_target_delimited_form`。
- **L1587 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Warning<"the delimited form of '#pragma omp declare target' without clauses is deprecated; use '#pragma omp begin declare target' instead">,`.
  **L1587 CN**: 继续一个多行参数列表、初始化器或聚合项：`Warning<"the delimited form of '#pragma omp declare target' without clauses is deprecated; use '#pragma omp begin declare target' instead">,`。
- **L1588 EN**: Adds a standalone statement or declaration: `InGroup<Deprecated>;`.
  **L1588 CN**: 添加一条独立语句或声明：`InGroup<Deprecated>;`。
- **L1589 EN**: Declares TableGen def record `err_omp_expected_clause`.
  **L1589 CN**: 声明 TableGen def 记录 `err_omp_expected_clause`。
- **L1590 EN**: Adds a standalone statement or declaration: `"expected at least one clause on '#pragma omp %0' directive">;`.
  **L1590 CN**: 添加一条独立语句或声明：`"expected at least one clause on '#pragma omp %0' directive">;`。
- **L1591 EN**: Declares TableGen def record `err_omp_mapper_illegal_identifier`.
  **L1591 CN**: 声明 TableGen def 记录 `err_omp_mapper_illegal_identifier`。
- **L1592 EN**: Adds a standalone statement or declaration: `"illegal OpenMP user-defined mapper identifier">;`.
  **L1592 CN**: 添加一条独立语句或声明：`"illegal OpenMP user-defined mapper identifier">;`。
- **L1593 EN**: Declares TableGen def record `err_omp_mapper_expected_declarator`.
  **L1593 CN**: 声明 TableGen def 记录 `err_omp_mapper_expected_declarator`。
- **L1594 EN**: Adds a standalone statement or declaration: `"expected declarator on 'omp declare mapper' directive">;`.
  **L1594 CN**: 添加一条独立语句或声明：`"expected declarator on 'omp declare mapper' directive">;`。
- **L1595 EN**: Declares TableGen def record `err_omp_unexpected_append_op`.
  **L1595 CN**: 声明 TableGen def 记录 `err_omp_unexpected_append_op`。
- **L1596 EN**: Adds a standalone statement or declaration: `"unexpected operation specified in 'append_args' clause, expected 'interop'">;`.
  **L1596 CN**: 添加一条独立语句或声明：`"unexpected operation specified in 'append_args' clause, expected 'interop'">;`。
- **L1597 EN**: Declares TableGen def record `err_omp_unexpected_execution_modifier`.
  **L1597 CN**: 声明 TableGen def 记录 `err_omp_unexpected_execution_modifier`。
- **L1598 EN**: Adds a standalone statement or declaration: `"unexpected 'execution' modifier in non-executable context">;`.
  **L1598 CN**: 添加一条独立语句或声明：`"unexpected 'execution' modifier in non-executable context">;`。
- **L1599 EN**: Declares TableGen def record `err_omp_unknown_adjust_args_op`.
  **L1599 CN**: 声明 TableGen def 记录 `err_omp_unknown_adjust_args_op`。
- **L1600 EN**: Continues the surrounding expression or declaration: `: Error<`.
  **L1600 CN**: 继续构造周围的表达式或声明：`: Error<`。
- **L1601 EN**: Continues the surrounding expression or declaration: `"incorrect 'adjust_args' type, expected 'need_device_ptr'%select{|, "`.
  **L1601 CN**: 继续构造周围的表达式或声明：`"incorrect 'adjust_args' type, expected 'need_device_ptr'%select{|, "`。
- **L1602 EN**: Adds a standalone statement or declaration: `"'need_device_addr',}0 or 'nothing'">;`.
  **L1602 CN**: 添加一条独立语句或声明：`"'need_device_addr',}0 or 'nothing'">;`。
- **L1603 EN**: Declares TableGen def record `err_omp_declare_variant_wrong_clause`.
  **L1603 CN**: 声明 TableGen def 记录 `err_omp_declare_variant_wrong_clause`。
- **L1604 EN**: Continues the surrounding expression or declaration: `"expected %select{'match'|'match', 'adjust_args', or 'append_args'}0 clause "`.
  **L1604 CN**: 继续构造周围的表达式或声明：`"expected %select{'match'|'match', 'adjust_args', or 'append_args'}0 clause "`。
- **L1605 EN**: Adds a standalone statement or declaration: `"on 'omp declare variant' directive">;`.
  **L1605 CN**: 添加一条独立语句或声明：`"on 'omp declare variant' directive">;`。
- **L1606 EN**: Declares TableGen def record `err_omp_non_by_ref_need_device_addr_modifier_argument`.
  **L1606 CN**: 声明 TableGen def 记录 `err_omp_non_by_ref_need_device_addr_modifier_argument`。
- **L1607 EN**: Continues the surrounding expression or declaration: `: Error<"expected reference type argument on 'adjust_args' clause with "`.
  **L1607 CN**: 继续构造周围的表达式或声明：`: Error<"expected reference type argument on 'adjust_args' clause with "`。
- **L1608 EN**: Adds a standalone statement or declaration: `"'need_device_addr' modifier">;`.
  **L1608 CN**: 添加一条独立语句或声明：`"'need_device_addr' modifier">;`。

### Lines 1609-1632

````tablegen
def err_omp_declare_variant_duplicate_nested_trait : Error<
  "nested OpenMP context selector contains duplicated trait '%0'"
  " in selector '%1' and set '%2' with different score">;
def err_omp_declare_variant_nested_user_condition : Error<
  "nested user conditions in OpenMP context selector not supported (yet)">;
def warn_omp_declare_variant_string_literal_or_identifier
    : Warning<"expected identifier or string literal describing a context "
              "%select{set|selector|property}0; "
              "%select{set|selector|property}0 skipped">,
      InGroup<OpenMPClauses>;
def warn_unknown_declare_variant_isa_trait
    : Warning<"isa trait '%0' is not known to the current target; verify the "
              "spelling or consider restricting the context selector with the "
              "'arch' selector further">,
      InGroup<SourceUsesOpenMP>;
def note_ompx_bare_clause : Note<
  "OpenMP extension clause '%0' only allowed with '#pragma omp %1'">;
def note_omp_declare_variant_ctx_options
    : Note<"context %select{set|selector|property}0 options are: %1">;
def warn_omp_declare_variant_expected
    : Warning<"expected '%0' after the %1; '%0' assumed">,
      InGroup<OpenMPClauses>;
def warn_omp_declare_variant_ctx_not_a_property
    : Warning<"'%0' is not a valid context property for the context selector "
````
- **L1609 EN**: Declares TableGen def record `err_omp_declare_variant_duplicate_nested_trait`.
  **L1609 CN**: 声明 TableGen def 记录 `err_omp_declare_variant_duplicate_nested_trait`。
- **L1610 EN**: Continues the surrounding expression or declaration: `"nested OpenMP context selector contains duplicated trait '%0'"`.
  **L1610 CN**: 继续构造周围的表达式或声明：`"nested OpenMP context selector contains duplicated trait '%0'"`。
- **L1611 EN**: Adds a standalone statement or declaration: `" in selector '%1' and set '%2' with different score">;`.
  **L1611 CN**: 添加一条独立语句或声明：`" in selector '%1' and set '%2' with different score">;`。
- **L1612 EN**: Declares TableGen def record `err_omp_declare_variant_nested_user_condition`.
  **L1612 CN**: 声明 TableGen def 记录 `err_omp_declare_variant_nested_user_condition`。
- **L1613 EN**: Executes a call or declaration centered on `supported`.
  **L1613 CN**: 执行以 `supported` 为核心的调用或声明。
- **L1614 EN**: Declares TableGen def record `warn_omp_declare_variant_string_literal_or_identifier`.
  **L1614 CN**: 声明 TableGen def 记录 `warn_omp_declare_variant_string_literal_or_identifier`。
- **L1615 EN**: Continues the surrounding expression or declaration: `: Warning<"expected identifier or string literal describing a context "`.
  **L1615 CN**: 继续构造周围的表达式或声明：`: Warning<"expected identifier or string literal describing a context "`。
- **L1616 EN**: Continues the surrounding expression or declaration: `"%select{set|selector|property}0; "`.
  **L1616 CN**: 继续构造周围的表达式或声明：`"%select{set|selector|property}0; "`。
- **L1617 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"%select{set|selector|property}0 skipped">,`.
  **L1617 CN**: 继续一个多行参数列表、初始化器或聚合项：`"%select{set|selector|property}0 skipped">,`。
- **L1618 EN**: Adds a standalone statement or declaration: `InGroup<OpenMPClauses>;`.
  **L1618 CN**: 添加一条独立语句或声明：`InGroup<OpenMPClauses>;`。
- **L1619 EN**: Declares TableGen def record `warn_unknown_declare_variant_isa_trait`.
  **L1619 CN**: 声明 TableGen def 记录 `warn_unknown_declare_variant_isa_trait`。
- **L1620 EN**: Continues the surrounding expression or declaration: `: Warning<"isa trait '%0' is not known to the current target; verify the "`.
  **L1620 CN**: 继续构造周围的表达式或声明：`: Warning<"isa trait '%0' is not known to the current target; verify the "`。
- **L1621 EN**: Continues the surrounding expression or declaration: `"spelling or consider restricting the context selector with the "`.
  **L1621 CN**: 继续构造周围的表达式或声明：`"spelling or consider restricting the context selector with the "`。
- **L1622 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'arch' selector further">,`.
  **L1622 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'arch' selector further">,`。
- **L1623 EN**: Adds a standalone statement or declaration: `InGroup<SourceUsesOpenMP>;`.
  **L1623 CN**: 添加一条独立语句或声明：`InGroup<SourceUsesOpenMP>;`。
- **L1624 EN**: Declares TableGen def record `note_ompx_bare_clause`.
  **L1624 CN**: 声明 TableGen def 记录 `note_ompx_bare_clause`。
- **L1625 EN**: Adds a standalone statement or declaration: `"OpenMP extension clause '%0' only allowed with '#pragma omp %1'">;`.
  **L1625 CN**: 添加一条独立语句或声明：`"OpenMP extension clause '%0' only allowed with '#pragma omp %1'">;`。
- **L1626 EN**: Declares TableGen def record `note_omp_declare_variant_ctx_options`.
  **L1626 CN**: 声明 TableGen def 记录 `note_omp_declare_variant_ctx_options`。
- **L1627 EN**: Adds a standalone statement or declaration: `: Note<"context %select{set|selector|property}0 options are: %1">;`.
  **L1627 CN**: 添加一条独立语句或声明：`: Note<"context %select{set|selector|property}0 options are: %1">;`。
- **L1628 EN**: Declares TableGen def record `warn_omp_declare_variant_expected`.
  **L1628 CN**: 声明 TableGen def 记录 `warn_omp_declare_variant_expected`。
- **L1629 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Warning<"expected '%0' after the %1; '%0' assumed">,`.
  **L1629 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Warning<"expected '%0' after the %1; '%0' assumed">,`。
- **L1630 EN**: Adds a standalone statement or declaration: `InGroup<OpenMPClauses>;`.
  **L1630 CN**: 添加一条独立语句或声明：`InGroup<OpenMPClauses>;`。
- **L1631 EN**: Declares TableGen def record `warn_omp_declare_variant_ctx_not_a_property`.
  **L1631 CN**: 声明 TableGen def 记录 `warn_omp_declare_variant_ctx_not_a_property`。
- **L1632 EN**: Continues the surrounding expression or declaration: `: Warning<"'%0' is not a valid context property for the context selector "`.
  **L1632 CN**: 继续构造周围的表达式或声明：`: Warning<"'%0' is not a valid context property for the context selector "`。

### Lines 1633-1656

````tablegen
              "'%1' and the context set '%2'; property ignored">,
      InGroup<OpenMPClauses>;
def note_omp_declare_variant_ctx_is_a
    : Note<"'%0' is a context %select{set|selector|property}1 not a context "
           "%select{set|selector|property}2">;
def note_omp_declare_variant_ctx_try : Note<"try 'match(%0={%1%2})'">;
def warn_omp_declare_variant_ctx_not_a_selector
    : Warning<"'%0' is not a valid context selector for the context set '%1'; "
              "selector ignored">,
      InGroup<OpenMPClauses>;
def warn_omp_declare_variant_ctx_not_a_set
    : Warning<"'%0' is not a valid context set in a `declare variant`; set "
              "ignored">,
      InGroup<OpenMPClauses>;
def warn_omp_declare_variant_ctx_mutiple_use
    : Warning<"the context %select{set|selector|property}0 '%1' was used "
              "already in the same 'omp declare variant' directive; "
              "%select{set|selector|property}0 ignored">,
      InGroup<OpenMPClauses>;
def note_omp_declare_variant_ctx_used_here
    : Note<"the previous context %select{set|selector|property}0 '%1' used "
           "here">;
def note_omp_declare_variant_ctx_continue_here
    : Note<"the ignored %select{set|selector|property}0 spans until here">;
````
- **L1633 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'%1' and the context set '%2'; property ignored">,`.
  **L1633 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'%1' and the context set '%2'; property ignored">,`。
- **L1634 EN**: Adds a standalone statement or declaration: `InGroup<OpenMPClauses>;`.
  **L1634 CN**: 添加一条独立语句或声明：`InGroup<OpenMPClauses>;`。
- **L1635 EN**: Declares TableGen def record `note_omp_declare_variant_ctx_is_a`.
  **L1635 CN**: 声明 TableGen def 记录 `note_omp_declare_variant_ctx_is_a`。
- **L1636 EN**: Continues the surrounding expression or declaration: `: Note<"'%0' is a context %select{set|selector|property}1 not a context "`.
  **L1636 CN**: 继续构造周围的表达式或声明：`: Note<"'%0' is a context %select{set|selector|property}1 not a context "`。
- **L1637 EN**: Adds a standalone statement or declaration: `"%select{set|selector|property}2">;`.
  **L1637 CN**: 添加一条独立语句或声明：`"%select{set|selector|property}2">;`。
- **L1638 EN**: Declares TableGen def record `note_omp_declare_variant_ctx_try`.
  **L1638 CN**: 声明 TableGen def 记录 `note_omp_declare_variant_ctx_try`。
- **L1639 EN**: Declares TableGen def record `warn_omp_declare_variant_ctx_not_a_selector`.
  **L1639 CN**: 声明 TableGen def 记录 `warn_omp_declare_variant_ctx_not_a_selector`。
- **L1640 EN**: Continues the surrounding expression or declaration: `: Warning<"'%0' is not a valid context selector for the context set '%1'; "`.
  **L1640 CN**: 继续构造周围的表达式或声明：`: Warning<"'%0' is not a valid context selector for the context set '%1'; "`。
- **L1641 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"selector ignored">,`.
  **L1641 CN**: 继续一个多行参数列表、初始化器或聚合项：`"selector ignored">,`。
- **L1642 EN**: Adds a standalone statement or declaration: `InGroup<OpenMPClauses>;`.
  **L1642 CN**: 添加一条独立语句或声明：`InGroup<OpenMPClauses>;`。
- **L1643 EN**: Declares TableGen def record `warn_omp_declare_variant_ctx_not_a_set`.
  **L1643 CN**: 声明 TableGen def 记录 `warn_omp_declare_variant_ctx_not_a_set`。
- **L1644 EN**: Continues the surrounding expression or declaration: `: Warning<"'%0' is not a valid context set in a `declare variant`; set "`.
  **L1644 CN**: 继续构造周围的表达式或声明：`: Warning<"'%0' is not a valid context set in a `declare variant`; set "`。
- **L1645 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"ignored">,`.
  **L1645 CN**: 继续一个多行参数列表、初始化器或聚合项：`"ignored">,`。
- **L1646 EN**: Adds a standalone statement or declaration: `InGroup<OpenMPClauses>;`.
  **L1646 CN**: 添加一条独立语句或声明：`InGroup<OpenMPClauses>;`。
- **L1647 EN**: Declares TableGen def record `warn_omp_declare_variant_ctx_mutiple_use`.
  **L1647 CN**: 声明 TableGen def 记录 `warn_omp_declare_variant_ctx_mutiple_use`。
- **L1648 EN**: Continues the surrounding expression or declaration: `: Warning<"the context %select{set|selector|property}0 '%1' was used "`.
  **L1648 CN**: 继续构造周围的表达式或声明：`: Warning<"the context %select{set|selector|property}0 '%1' was used "`。
- **L1649 EN**: Continues the surrounding expression or declaration: `"already in the same 'omp declare variant' directive; "`.
  **L1649 CN**: 继续构造周围的表达式或声明：`"already in the same 'omp declare variant' directive; "`。
- **L1650 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"%select{set|selector|property}0 ignored">,`.
  **L1650 CN**: 继续一个多行参数列表、初始化器或聚合项：`"%select{set|selector|property}0 ignored">,`。
- **L1651 EN**: Adds a standalone statement or declaration: `InGroup<OpenMPClauses>;`.
  **L1651 CN**: 添加一条独立语句或声明：`InGroup<OpenMPClauses>;`。
- **L1652 EN**: Declares TableGen def record `note_omp_declare_variant_ctx_used_here`.
  **L1652 CN**: 声明 TableGen def 记录 `note_omp_declare_variant_ctx_used_here`。
- **L1653 EN**: Continues the surrounding expression or declaration: `: Note<"the previous context %select{set|selector|property}0 '%1' used "`.
  **L1653 CN**: 继续构造周围的表达式或声明：`: Note<"the previous context %select{set|selector|property}0 '%1' used "`。
- **L1654 EN**: Adds a standalone statement or declaration: `"here">;`.
  **L1654 CN**: 添加一条独立语句或声明：`"here">;`。
- **L1655 EN**: Declares TableGen def record `note_omp_declare_variant_ctx_continue_here`.
  **L1655 CN**: 声明 TableGen def 记录 `note_omp_declare_variant_ctx_continue_here`。
- **L1656 EN**: Adds a standalone statement or declaration: `: Note<"the ignored %select{set|selector|property}0 spans until here">;`.
  **L1656 CN**: 添加一条独立语句或声明：`: Note<"the ignored %select{set|selector|property}0 spans until here">;`。

### Lines 1657-1680

````tablegen
def warn_omp_ctx_incompatible_selector_for_set
    : Warning<"the context selector '%0' is not valid for the context set "
              "'%1'; selector ignored">,
      InGroup<OpenMPClauses>;
def note_omp_ctx_compatible_set_for_selector
    : Note<"the context selector '%0' can be nested in the context set '%1'; "
           "try 'match(%1={%0%select{|(property)}2})'">;
def warn_omp_ctx_selector_without_properties
    : Warning<"the context selector '%0' in context set '%1' requires a "
              "context property defined in parentheses; selector ignored">,
      InGroup<OpenMPClauses>;
def warn_omp_ctx_incompatible_property_for_selector
    : Warning<"the context property '%0' is not valid for the context selector "
              "'%1' and the context set '%2'; property ignored">,
      InGroup<OpenMPClauses>;
def note_omp_ctx_compatible_set_and_selector_for_property
    : Note<"the context property '%0' can be nested in the context selector "
           "'%1' which is nested in the context set '%2'; try "
           "'match(%2={%1(%0)})'">;
def warn_omp_ctx_incompatible_score_for_property
    : Warning<"the context selector '%0' in the context set '%1' cannot have a "
              "score (%quoted2); score ignored">,
      InGroup<OpenMPClauses>;
def warn_omp_more_one_device_type_clause
````
- **L1657 EN**: Declares TableGen def record `warn_omp_ctx_incompatible_selector_for_set`.
  **L1657 CN**: 声明 TableGen def 记录 `warn_omp_ctx_incompatible_selector_for_set`。
- **L1658 EN**: Continues the surrounding expression or declaration: `: Warning<"the context selector '%0' is not valid for the context set "`.
  **L1658 CN**: 继续构造周围的表达式或声明：`: Warning<"the context selector '%0' is not valid for the context set "`。
- **L1659 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'%1'; selector ignored">,`.
  **L1659 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'%1'; selector ignored">,`。
- **L1660 EN**: Adds a standalone statement or declaration: `InGroup<OpenMPClauses>;`.
  **L1660 CN**: 添加一条独立语句或声明：`InGroup<OpenMPClauses>;`。
- **L1661 EN**: Declares TableGen def record `note_omp_ctx_compatible_set_for_selector`.
  **L1661 CN**: 声明 TableGen def 记录 `note_omp_ctx_compatible_set_for_selector`。
- **L1662 EN**: Continues the surrounding expression or declaration: `: Note<"the context selector '%0' can be nested in the context set '%1'; "`.
  **L1662 CN**: 继续构造周围的表达式或声明：`: Note<"the context selector '%0' can be nested in the context set '%1'; "`。
- **L1663 EN**: Executes a call or declaration centered on `'match`.
  **L1663 CN**: 执行以 `'match` 为核心的调用或声明。
- **L1664 EN**: Declares TableGen def record `warn_omp_ctx_selector_without_properties`.
  **L1664 CN**: 声明 TableGen def 记录 `warn_omp_ctx_selector_without_properties`。
- **L1665 EN**: Continues the surrounding expression or declaration: `: Warning<"the context selector '%0' in context set '%1' requires a "`.
  **L1665 CN**: 继续构造周围的表达式或声明：`: Warning<"the context selector '%0' in context set '%1' requires a "`。
- **L1666 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"context property defined in parentheses; selector ignored">,`.
  **L1666 CN**: 继续一个多行参数列表、初始化器或聚合项：`"context property defined in parentheses; selector ignored">,`。
- **L1667 EN**: Adds a standalone statement or declaration: `InGroup<OpenMPClauses>;`.
  **L1667 CN**: 添加一条独立语句或声明：`InGroup<OpenMPClauses>;`。
- **L1668 EN**: Declares TableGen def record `warn_omp_ctx_incompatible_property_for_selector`.
  **L1668 CN**: 声明 TableGen def 记录 `warn_omp_ctx_incompatible_property_for_selector`。
- **L1669 EN**: Continues the surrounding expression or declaration: `: Warning<"the context property '%0' is not valid for the context selector "`.
  **L1669 CN**: 继续构造周围的表达式或声明：`: Warning<"the context property '%0' is not valid for the context selector "`。
- **L1670 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'%1' and the context set '%2'; property ignored">,`.
  **L1670 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'%1' and the context set '%2'; property ignored">,`。
- **L1671 EN**: Adds a standalone statement or declaration: `InGroup<OpenMPClauses>;`.
  **L1671 CN**: 添加一条独立语句或声明：`InGroup<OpenMPClauses>;`。
- **L1672 EN**: Declares TableGen def record `note_omp_ctx_compatible_set_and_selector_for_property`.
  **L1672 CN**: 声明 TableGen def 记录 `note_omp_ctx_compatible_set_and_selector_for_property`。
- **L1673 EN**: Continues the surrounding expression or declaration: `: Note<"the context property '%0' can be nested in the context selector "`.
  **L1673 CN**: 继续构造周围的表达式或声明：`: Note<"the context property '%0' can be nested in the context selector "`。
- **L1674 EN**: Continues the surrounding expression or declaration: `"'%1' which is nested in the context set '%2'; try "`.
  **L1674 CN**: 继续构造周围的表达式或声明：`"'%1' which is nested in the context set '%2'; try "`。
- **L1675 EN**: Executes a call or declaration centered on `"'match`.
  **L1675 CN**: 执行以 `"'match` 为核心的调用或声明。
- **L1676 EN**: Declares TableGen def record `warn_omp_ctx_incompatible_score_for_property`.
  **L1676 CN**: 声明 TableGen def 记录 `warn_omp_ctx_incompatible_score_for_property`。
- **L1677 EN**: Continues the surrounding expression or declaration: `: Warning<"the context selector '%0' in the context set '%1' cannot have a "`.
  **L1677 CN**: 继续构造周围的表达式或声明：`: Warning<"the context selector '%0' in the context set '%1' cannot have a "`。
- **L1678 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"score (%quoted2); score ignored">,`.
  **L1678 CN**: 继续一个多行参数列表、初始化器或聚合项：`"score (%quoted2); score ignored">,`。
- **L1679 EN**: Adds a standalone statement or declaration: `InGroup<OpenMPClauses>;`.
  **L1679 CN**: 添加一条独立语句或声明：`InGroup<OpenMPClauses>;`。
- **L1680 EN**: Declares TableGen def record `warn_omp_more_one_device_type_clause`.
  **L1680 CN**: 声明 TableGen def 记录 `warn_omp_more_one_device_type_clause`。

### Lines 1681-1704

````tablegen
    : Warning<"more than one 'device_type' clause is specified">,
      InGroup<OpenMPClauses>;
def err_omp_variant_ctx_second_match_extension : Error<
  "only a single match extension allowed per OpenMP context selector">;
def err_omp_invalid_dsa: Error<
  "data-sharing attribute '%0' in '%1' clause requires OpenMP version %2 or above">;
def err_omp_expected_punc_after_interop_mod : Error<
  "expected ',' after interop modifier">;
def err_omp_expected_interop_type : Error<
  "expected interop type: 'target' and/or 'targetsync'">;
def warn_omp_more_one_interop_type
  : Warning<"interop type '%0' cannot be specified more than once">,
    InGroup<OpenMPClauses>;
def err_expected_sequence_or_directive : Error<
  "expected an OpenMP 'directive' or 'sequence' attribute argument">;
def ext_omp_attributes : ExtWarn<
  "specifying OpenMP directives with [[]] is an OpenMP 5.1 extension">,
  InGroup<OpenMP51Ext>;
def warn_omp51_compat_attributes : Warning<
  "specifying OpenMP directives with [[]] is incompatible with OpenMP "
  "standards before OpenMP 5.1">,
  InGroup<OpenMPPre51Compat>, DefaultIgnore;
def err_omp_expected_colon : Error<"missing ':' in %0">;
def err_omp_missing_comma : Error< "missing ',' after %0">;
````
- **L1681 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Warning<"more than one 'device_type' clause is specified">,`.
  **L1681 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Warning<"more than one 'device_type' clause is specified">,`。
- **L1682 EN**: Adds a standalone statement or declaration: `InGroup<OpenMPClauses>;`.
  **L1682 CN**: 添加一条独立语句或声明：`InGroup<OpenMPClauses>;`。
- **L1683 EN**: Declares TableGen def record `err_omp_variant_ctx_second_match_extension`.
  **L1683 CN**: 声明 TableGen def 记录 `err_omp_variant_ctx_second_match_extension`。
- **L1684 EN**: Adds a standalone statement or declaration: `"only a single match extension allowed per OpenMP context selector">;`.
  **L1684 CN**: 添加一条独立语句或声明：`"only a single match extension allowed per OpenMP context selector">;`。
- **L1685 EN**: Declares TableGen def record `err_omp_invalid_dsa`.
  **L1685 CN**: 声明 TableGen def 记录 `err_omp_invalid_dsa`。
- **L1686 EN**: Adds a standalone statement or declaration: `"data-sharing attribute '%0' in '%1' clause requires OpenMP version %2 or above">;`.
  **L1686 CN**: 添加一条独立语句或声明：`"data-sharing attribute '%0' in '%1' clause requires OpenMP version %2 or above">;`。
- **L1687 EN**: Declares TableGen def record `err_omp_expected_punc_after_interop_mod`.
  **L1687 CN**: 声明 TableGen def 记录 `err_omp_expected_punc_after_interop_mod`。
- **L1688 EN**: Adds a standalone statement or declaration: `"expected ',' after interop modifier">;`.
  **L1688 CN**: 添加一条独立语句或声明：`"expected ',' after interop modifier">;`。
- **L1689 EN**: Declares TableGen def record `err_omp_expected_interop_type`.
  **L1689 CN**: 声明 TableGen def 记录 `err_omp_expected_interop_type`。
- **L1690 EN**: Adds a standalone statement or declaration: `"expected interop type: 'target' and/or 'targetsync'">;`.
  **L1690 CN**: 添加一条独立语句或声明：`"expected interop type: 'target' and/or 'targetsync'">;`。
- **L1691 EN**: Declares TableGen def record `warn_omp_more_one_interop_type`.
  **L1691 CN**: 声明 TableGen def 记录 `warn_omp_more_one_interop_type`。
- **L1692 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Warning<"interop type '%0' cannot be specified more than once">,`.
  **L1692 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Warning<"interop type '%0' cannot be specified more than once">,`。
- **L1693 EN**: Adds a standalone statement or declaration: `InGroup<OpenMPClauses>;`.
  **L1693 CN**: 添加一条独立语句或声明：`InGroup<OpenMPClauses>;`。
- **L1694 EN**: Declares TableGen def record `err_expected_sequence_or_directive`.
  **L1694 CN**: 声明 TableGen def 记录 `err_expected_sequence_or_directive`。
- **L1695 EN**: Adds a standalone statement or declaration: `"expected an OpenMP 'directive' or 'sequence' attribute argument">;`.
  **L1695 CN**: 添加一条独立语句或声明：`"expected an OpenMP 'directive' or 'sequence' attribute argument">;`。
- **L1696 EN**: Declares TableGen def record `ext_omp_attributes`.
  **L1696 CN**: 声明 TableGen def 记录 `ext_omp_attributes`。
- **L1697 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"specifying OpenMP directives with [[]] is an OpenMP 5.1 extension">,`.
  **L1697 CN**: 继续一个多行参数列表、初始化器或聚合项：`"specifying OpenMP directives with [[]] is an OpenMP 5.1 extension">,`。
- **L1698 EN**: Adds a standalone statement or declaration: `InGroup<OpenMP51Ext>;`.
  **L1698 CN**: 添加一条独立语句或声明：`InGroup<OpenMP51Ext>;`。
- **L1699 EN**: Declares TableGen def record `warn_omp51_compat_attributes`.
  **L1699 CN**: 声明 TableGen def 记录 `warn_omp51_compat_attributes`。
- **L1700 EN**: Continues the surrounding expression or declaration: `"specifying OpenMP directives with [[]] is incompatible with OpenMP "`.
  **L1700 CN**: 继续构造周围的表达式或声明：`"specifying OpenMP directives with [[]] is incompatible with OpenMP "`。
- **L1701 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"standards before OpenMP 5.1">,`.
  **L1701 CN**: 继续一个多行参数列表、初始化器或聚合项：`"standards before OpenMP 5.1">,`。
- **L1702 EN**: Adds a standalone statement or declaration: `InGroup<OpenMPPre51Compat>, DefaultIgnore;`.
  **L1702 CN**: 添加一条独立语句或声明：`InGroup<OpenMPPre51Compat>, DefaultIgnore;`。
- **L1703 EN**: Declares TableGen def record `err_omp_expected_colon`.
  **L1703 CN**: 声明 TableGen def 记录 `err_omp_expected_colon`。
- **L1704 EN**: Declares TableGen def record `err_omp_missing_comma`.
  **L1704 CN**: 声明 TableGen def 记录 `err_omp_missing_comma`。

### Lines 1705-1728

````tablegen
def err_omp_expected_context_selector
    : Error<"expected valid context selector in %0">;
def warn_omp_default_deprecated : Warning<"'default' clause for"
  " 'metadirective' is deprecated; use 'otherwise' instead">, InGroup<Deprecated>;
def err_omp_requires_out_inout_depend_type : Error<
  "reserved locator 'omp_all_memory' requires 'out' or 'inout' "
  "dependency types">;
def warn_omp_more_one_omp_all_memory : Warning<
  "reserved locator 'omp_all_memory' cannot be specified more than once">,
  InGroup<OpenMPClauses>;
def warn_omp_depend_in_ordered_deprecated : Warning<"'depend' clause for"
  " 'ordered' is deprecated; use 'doacross' instead">, InGroup<Deprecated>;
def warn_omp_invalid_attribute_for_ompx_attributes : Warning<"'ompx_attribute' clause only allows "
  "'amdgpu_flat_work_group_size', 'amdgpu_waves_per_eu', and 'launch_bounds'; "
  "%0 is ignored">, InGroup<OpenMPExtensions>;
def err_omp_duplicate_modifier : Error<"duplicate modifier '%0' in '%1' clause">;
def err_omp_expected_modifier : Error<"expected modifier in '%0' clause">;
def err_omp_unknown_need_device_ptr_kind
    : Error<
        "invalid argument for 'need_device_ptr' kind in 'adjust_args' clause; "
        "expected 'fb_nullify' or 'fb_preserve'">;

// Pragma loop support.
def err_pragma_loop_missing_argument : Error<
````
- **L1705 EN**: Declares TableGen def record `err_omp_expected_context_selector`.
  **L1705 CN**: 声明 TableGen def 记录 `err_omp_expected_context_selector`。
- **L1706 EN**: Adds a standalone statement or declaration: `: Error<"expected valid context selector in %0">;`.
  **L1706 CN**: 添加一条独立语句或声明：`: Error<"expected valid context selector in %0">;`。
- **L1707 EN**: Declares TableGen def record `warn_omp_default_deprecated`.
  **L1707 CN**: 声明 TableGen def 记录 `warn_omp_default_deprecated`。
- **L1708 EN**: Adds a standalone statement or declaration: `" 'metadirective' is deprecated; use 'otherwise' instead">, InGroup<Deprecated>;`.
  **L1708 CN**: 添加一条独立语句或声明：`" 'metadirective' is deprecated; use 'otherwise' instead">, InGroup<Deprecated>;`。
- **L1709 EN**: Declares TableGen def record `err_omp_requires_out_inout_depend_type`.
  **L1709 CN**: 声明 TableGen def 记录 `err_omp_requires_out_inout_depend_type`。
- **L1710 EN**: Continues the surrounding expression or declaration: `"reserved locator 'omp_all_memory' requires 'out' or 'inout' "`.
  **L1710 CN**: 继续构造周围的表达式或声明：`"reserved locator 'omp_all_memory' requires 'out' or 'inout' "`。
- **L1711 EN**: Adds a standalone statement or declaration: `"dependency types">;`.
  **L1711 CN**: 添加一条独立语句或声明：`"dependency types">;`。
- **L1712 EN**: Declares TableGen def record `warn_omp_more_one_omp_all_memory`.
  **L1712 CN**: 声明 TableGen def 记录 `warn_omp_more_one_omp_all_memory`。
- **L1713 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"reserved locator 'omp_all_memory' cannot be specified more than once">,`.
  **L1713 CN**: 继续一个多行参数列表、初始化器或聚合项：`"reserved locator 'omp_all_memory' cannot be specified more than once">,`。
- **L1714 EN**: Adds a standalone statement or declaration: `InGroup<OpenMPClauses>;`.
  **L1714 CN**: 添加一条独立语句或声明：`InGroup<OpenMPClauses>;`。
- **L1715 EN**: Declares TableGen def record `warn_omp_depend_in_ordered_deprecated`.
  **L1715 CN**: 声明 TableGen def 记录 `warn_omp_depend_in_ordered_deprecated`。
- **L1716 EN**: Adds a standalone statement or declaration: `" 'ordered' is deprecated; use 'doacross' instead">, InGroup<Deprecated>;`.
  **L1716 CN**: 添加一条独立语句或声明：`" 'ordered' is deprecated; use 'doacross' instead">, InGroup<Deprecated>;`。
- **L1717 EN**: Declares TableGen def record `warn_omp_invalid_attribute_for_ompx_attributes`.
  **L1717 CN**: 声明 TableGen def 记录 `warn_omp_invalid_attribute_for_ompx_attributes`。
- **L1718 EN**: Continues the surrounding expression or declaration: `"'amdgpu_flat_work_group_size', 'amdgpu_waves_per_eu', and 'launch_bounds'; "`.
  **L1718 CN**: 继续构造周围的表达式或声明：`"'amdgpu_flat_work_group_size', 'amdgpu_waves_per_eu', and 'launch_bounds'; "`。
- **L1719 EN**: Adds a standalone statement or declaration: `"%0 is ignored">, InGroup<OpenMPExtensions>;`.
  **L1719 CN**: 添加一条独立语句或声明：`"%0 is ignored">, InGroup<OpenMPExtensions>;`。
- **L1720 EN**: Declares TableGen def record `err_omp_duplicate_modifier`.
  **L1720 CN**: 声明 TableGen def 记录 `err_omp_duplicate_modifier`。
- **L1721 EN**: Declares TableGen def record `err_omp_expected_modifier`.
  **L1721 CN**: 声明 TableGen def 记录 `err_omp_expected_modifier`。
- **L1722 EN**: Declares TableGen def record `err_omp_unknown_need_device_ptr_kind`.
  **L1722 CN**: 声明 TableGen def 记录 `err_omp_unknown_need_device_ptr_kind`。
- **L1723 EN**: Continues the surrounding expression or declaration: `: Error<`.
  **L1723 CN**: 继续构造周围的表达式或声明：`: Error<`。
- **L1724 EN**: Continues the surrounding expression or declaration: `"invalid argument for 'need_device_ptr' kind in 'adjust_args' clause; "`.
  **L1724 CN**: 继续构造周围的表达式或声明：`"invalid argument for 'need_device_ptr' kind in 'adjust_args' clause; "`。
- **L1725 EN**: Adds a standalone statement or declaration: `"expected 'fb_nullify' or 'fb_preserve'">;`.
  **L1725 CN**: 添加一条独立语句或声明：`"expected 'fb_nullify' or 'fb_preserve'">;`。
- **L1726 EN**: Blank line separating nearby declarations or logic blocks.
  **L1726 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1727 EN**: Comment explains nearby logic, constraints, or intent: `Pragma loop support.`.
  **L1727 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Pragma loop support.`。
- **L1728 EN**: Declares TableGen def record `err_pragma_loop_missing_argument`.
  **L1728 CN**: 声明 TableGen def 记录 `err_pragma_loop_missing_argument`。

### Lines 1729-1752

````tablegen
  "missing argument; expected %select{an integer value|"
  "'enable'%select{|, 'full'}1%select{|, 'assume_safety'}2 or 'disable'}0">;
def err_pragma_loop_invalid_option : Error<
  "%select{invalid|missing}0 option%select{ %1|}0; expected vectorize, "
  "vectorize_width, interleave, interleave_count, unroll, unroll_count, "
  "pipeline, pipeline_initiation_interval, vectorize_predicate, or distribute">;
def err_pragma_loop_invalid_vectorize_option : Error<
  "vectorize_width loop hint malformed; use vectorize_width(X, fixed) or "
  "vectorize_width(X, scalable) where X is an integer, or vectorize_width('fixed' or 'scalable')">;
def note_pragma_loop_invalid_vectorize_option : Note<
  "vectorize_width loop hint malformed; use vectorize_width(X, fixed) or "
  "vectorize_width(X, scalable) where X is an integer, or vectorize_width('fixed' or 'scalable')">;

def err_pragma_fp_invalid_option : Error<
  "%select{invalid|missing}0 option%select{ %1|}0; expected 'contract', 'reassociate', 'reciprocal', or 'exceptions'">;
def err_pragma_fp_invalid_argument : Error<
  "unexpected argument '%0' to '#pragma clang fp %1'; expected "
  "%select{"
  "'fast' or 'on' or 'off'|"
  "'on' or 'off'|"
  "'on' or 'off'|"
  "'ignore', 'maytrap' or 'strict'|"
  "'source', 'double' or 'extended'}2">;

````
- **L1729 EN**: Continues the surrounding expression or declaration: `"missing argument; expected %select{an integer value|"`.
  **L1729 CN**: 继续构造周围的表达式或声明：`"missing argument; expected %select{an integer value|"`。
- **L1730 EN**: Adds a standalone statement or declaration: `"'enable'%select{|, 'full'}1%select{|, 'assume_safety'}2 or 'disable'}0">;`.
  **L1730 CN**: 添加一条独立语句或声明：`"'enable'%select{|, 'full'}1%select{|, 'assume_safety'}2 or 'disable'}0">;`。
- **L1731 EN**: Declares TableGen def record `err_pragma_loop_invalid_option`.
  **L1731 CN**: 声明 TableGen def 记录 `err_pragma_loop_invalid_option`。
- **L1732 EN**: Continues the surrounding expression or declaration: `"%select{invalid|missing}0 option%select{ %1|}0; expected vectorize, "`.
  **L1732 CN**: 继续构造周围的表达式或声明：`"%select{invalid|missing}0 option%select{ %1|}0; expected vectorize, "`。
- **L1733 EN**: Continues the surrounding expression or declaration: `"vectorize_width, interleave, interleave_count, unroll, unroll_count, "`.
  **L1733 CN**: 继续构造周围的表达式或声明：`"vectorize_width, interleave, interleave_count, unroll, unroll_count, "`。
- **L1734 EN**: Adds a standalone statement or declaration: `"pipeline, pipeline_initiation_interval, vectorize_predicate, or distribute">;`.
  **L1734 CN**: 添加一条独立语句或声明：`"pipeline, pipeline_initiation_interval, vectorize_predicate, or distribute">;`。
- **L1735 EN**: Declares TableGen def record `err_pragma_loop_invalid_vectorize_option`.
  **L1735 CN**: 声明 TableGen def 记录 `err_pragma_loop_invalid_vectorize_option`。
- **L1736 EN**: Continues logic associated with callable symbol `vectorize_width`.
  **L1736 CN**: 继续与可调用符号 `vectorize_width` 相关的逻辑。
- **L1737 EN**: Executes a call or declaration centered on `"vectorize_width`.
  **L1737 CN**: 执行以 `"vectorize_width` 为核心的调用或声明。
- **L1738 EN**: Declares TableGen def record `note_pragma_loop_invalid_vectorize_option`.
  **L1738 CN**: 声明 TableGen def 记录 `note_pragma_loop_invalid_vectorize_option`。
- **L1739 EN**: Continues logic associated with callable symbol `vectorize_width`.
  **L1739 CN**: 继续与可调用符号 `vectorize_width` 相关的逻辑。
- **L1740 EN**: Executes a call or declaration centered on `"vectorize_width`.
  **L1740 CN**: 执行以 `"vectorize_width` 为核心的调用或声明。
- **L1741 EN**: Blank line separating nearby declarations or logic blocks.
  **L1741 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1742 EN**: Declares TableGen def record `err_pragma_fp_invalid_option`.
  **L1742 CN**: 声明 TableGen def 记录 `err_pragma_fp_invalid_option`。
- **L1743 EN**: Adds a standalone statement or declaration: `"%select{invalid|missing}0 option%select{ %1|}0; expected 'contract', 'reassociate', 'reciprocal', or 'exceptions'">;`.
  **L1743 CN**: 添加一条独立语句或声明：`"%select{invalid|missing}0 option%select{ %1|}0; expected 'contract', 'reassociate', 'reciprocal', or 'exceptions'">;`。
- **L1744 EN**: Declares TableGen def record `err_pragma_fp_invalid_argument`.
  **L1744 CN**: 声明 TableGen def 记录 `err_pragma_fp_invalid_argument`。
- **L1745 EN**: Continues the surrounding expression or declaration: `"unexpected argument '%0' to '#pragma clang fp %1'; expected "`.
  **L1745 CN**: 继续构造周围的表达式或声明：`"unexpected argument '%0' to '#pragma clang fp %1'; expected "`。
- **L1746 EN**: Continues the surrounding expression or declaration: `"%select{"`.
  **L1746 CN**: 继续构造周围的表达式或声明：`"%select{"`。
- **L1747 EN**: Continues the surrounding expression or declaration: `"'fast' or 'on' or 'off'|"`.
  **L1747 CN**: 继续构造周围的表达式或声明：`"'fast' or 'on' or 'off'|"`。
- **L1748 EN**: Continues the surrounding expression or declaration: `"'on' or 'off'|"`.
  **L1748 CN**: 继续构造周围的表达式或声明：`"'on' or 'off'|"`。
- **L1749 EN**: Continues the surrounding expression or declaration: `"'on' or 'off'|"`.
  **L1749 CN**: 继续构造周围的表达式或声明：`"'on' or 'off'|"`。
- **L1750 EN**: Continues the surrounding expression or declaration: `"'ignore', 'maytrap' or 'strict'|"`.
  **L1750 CN**: 继续构造周围的表达式或声明：`"'ignore', 'maytrap' or 'strict'|"`。
- **L1751 EN**: Adds a standalone statement or declaration: `"'source', 'double' or 'extended'}2">;`.
  **L1751 CN**: 添加一条独立语句或声明：`"'source', 'double' or 'extended'}2">;`。
- **L1752 EN**: Blank line separating nearby declarations or logic blocks.
  **L1752 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1753-1776

````tablegen
def err_pragma_invalid_keyword : Error<
  "invalid argument; expected 'enable'%select{|, 'full'}0%select{|, 'assume_safety'}1 or 'disable'">;
def err_pragma_pipeline_invalid_keyword : Error<
    "invalid argument; expected 'disable'">;

// API notes.
def err_type_unparsed : Error<"unparsed tokens following type">;

// Pragma unroll support.
def warn_pragma_unroll_cuda_value_in_parens : Warning<
  "argument to '#pragma unroll' should not be in parentheses in CUDA C/C++">,
  InGroup<CudaCompat>;

def err_ptrauth_qualifier_bad_arg_count : Error<
  "'__ptrauth' qualifier must take between 1 and 3 arguments">;

def warn_cuda_attr_lambda_position : Warning<
  "nvcc does not allow '__%0__' to appear after the parameter list in lambdas">,
  InGroup<CudaCompat>;
def warn_pragma_force_cuda_host_device_bad_arg : Warning<
  "incorrect use of #pragma clang force_cuda_host_device begin|end">,
  InGroup<IgnoredPragmas>;
def err_pragma_cannot_end_force_cuda_host_device : Error<
  "force_cuda_host_device end pragma without matching "
````
- **L1753 EN**: Declares TableGen def record `err_pragma_invalid_keyword`.
  **L1753 CN**: 声明 TableGen def 记录 `err_pragma_invalid_keyword`。
- **L1754 EN**: Adds a standalone statement or declaration: `"invalid argument; expected 'enable'%select{|, 'full'}0%select{|, 'assume_safety'}1 or 'disable'">;`.
  **L1754 CN**: 添加一条独立语句或声明：`"invalid argument; expected 'enable'%select{|, 'full'}0%select{|, 'assume_safety'}1 or 'disable'">;`。
- **L1755 EN**: Declares TableGen def record `err_pragma_pipeline_invalid_keyword`.
  **L1755 CN**: 声明 TableGen def 记录 `err_pragma_pipeline_invalid_keyword`。
- **L1756 EN**: Adds a standalone statement or declaration: `"invalid argument; expected 'disable'">;`.
  **L1756 CN**: 添加一条独立语句或声明：`"invalid argument; expected 'disable'">;`。
- **L1757 EN**: Blank line separating nearby declarations or logic blocks.
  **L1757 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1758 EN**: Comment explains nearby logic, constraints, or intent: `API notes.`.
  **L1758 CN**: 注释解释附近代码的逻辑、约束或设计意图：`API notes.`。
- **L1759 EN**: Declares TableGen def record `err_type_unparsed`.
  **L1759 CN**: 声明 TableGen def 记录 `err_type_unparsed`。
- **L1760 EN**: Blank line separating nearby declarations or logic blocks.
  **L1760 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1761 EN**: Comment explains nearby logic, constraints, or intent: `Pragma unroll support.`.
  **L1761 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Pragma unroll support.`。
- **L1762 EN**: Declares TableGen def record `warn_pragma_unroll_cuda_value_in_parens`.
  **L1762 CN**: 声明 TableGen def 记录 `warn_pragma_unroll_cuda_value_in_parens`。
- **L1763 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"argument to '#pragma unroll' should not be in parentheses in CUDA C/C++">,`.
  **L1763 CN**: 继续一个多行参数列表、初始化器或聚合项：`"argument to '#pragma unroll' should not be in parentheses in CUDA C/C++">,`。
- **L1764 EN**: Adds a standalone statement or declaration: `InGroup<CudaCompat>;`.
  **L1764 CN**: 添加一条独立语句或声明：`InGroup<CudaCompat>;`。
- **L1765 EN**: Blank line separating nearby declarations or logic blocks.
  **L1765 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1766 EN**: Declares TableGen def record `err_ptrauth_qualifier_bad_arg_count`.
  **L1766 CN**: 声明 TableGen def 记录 `err_ptrauth_qualifier_bad_arg_count`。
- **L1767 EN**: Adds a standalone statement or declaration: `"'__ptrauth' qualifier must take between 1 and 3 arguments">;`.
  **L1767 CN**: 添加一条独立语句或声明：`"'__ptrauth' qualifier must take between 1 and 3 arguments">;`。
- **L1768 EN**: Blank line separating nearby declarations or logic blocks.
  **L1768 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1769 EN**: Declares TableGen def record `warn_cuda_attr_lambda_position`.
  **L1769 CN**: 声明 TableGen def 记录 `warn_cuda_attr_lambda_position`。
- **L1770 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"nvcc does not allow '__%0__' to appear after the parameter list in lambdas">,`.
  **L1770 CN**: 继续一个多行参数列表、初始化器或聚合项：`"nvcc does not allow '__%0__' to appear after the parameter list in lambdas">,`。
- **L1771 EN**: Adds a standalone statement or declaration: `InGroup<CudaCompat>;`.
  **L1771 CN**: 添加一条独立语句或声明：`InGroup<CudaCompat>;`。
- **L1772 EN**: Declares TableGen def record `warn_pragma_force_cuda_host_device_bad_arg`.
  **L1772 CN**: 声明 TableGen def 记录 `warn_pragma_force_cuda_host_device_bad_arg`。
- **L1773 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"incorrect use of #pragma clang force_cuda_host_device begin|end">,`.
  **L1773 CN**: 继续一个多行参数列表、初始化器或聚合项：`"incorrect use of #pragma clang force_cuda_host_device begin|end">,`。
- **L1774 EN**: Adds a standalone statement or declaration: `InGroup<IgnoredPragmas>;`.
  **L1774 CN**: 添加一条独立语句或声明：`InGroup<IgnoredPragmas>;`。
- **L1775 EN**: Declares TableGen def record `err_pragma_cannot_end_force_cuda_host_device`.
  **L1775 CN**: 声明 TableGen def 记录 `err_pragma_cannot_end_force_cuda_host_device`。
- **L1776 EN**: Continues the surrounding expression or declaration: `"force_cuda_host_device end pragma without matching "`.
  **L1776 CN**: 继续构造周围的表达式或声明：`"force_cuda_host_device end pragma without matching "`。

### Lines 1777-1800

````tablegen
  "force_cuda_host_device begin">;

def warn_ext_int_deprecated : Warning<
  "'_ExtInt' is deprecated; use '_BitInt' instead">, InGroup<DeprecatedType>;
def ext_bit_int : Extension<
  "'_BitInt' in %select{C17 and earlier|C++}0 is a Clang extension">,
  InGroup<BitIntExtension>;
} // end of Parse Issue category.

let CategoryName = "Modules Issue" in {
def err_unexpected_module_or_import_decl : Error<
  "%select{module|import}0 declaration can only appear at the top level">;
def err_attribute_not_module_attr : Error<
  "%0 attribute cannot be applied to a module">;
def err_keyword_not_module_attr : Error<
  "%0 cannot be applied to a module">;
def err_attribute_not_import_attr : Error<
  "%0 attribute cannot be applied to a module import">;
def err_keyword_not_import_attr : Error<
  "%0 cannot be applied to a module import">;
def err_module_expected_semi : Error<
  "expected ';' after module name">;
def err_expected_semi_after_module_or_import
  : Error<"%0 directive must end with a ';'">;
````
- **L1777 EN**: Adds a standalone statement or declaration: `"force_cuda_host_device begin">;`.
  **L1777 CN**: 添加一条独立语句或声明：`"force_cuda_host_device begin">;`。
- **L1778 EN**: Blank line separating nearby declarations or logic blocks.
  **L1778 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1779 EN**: Declares TableGen def record `warn_ext_int_deprecated`.
  **L1779 CN**: 声明 TableGen def 记录 `warn_ext_int_deprecated`。
- **L1780 EN**: Adds a standalone statement or declaration: `"'_ExtInt' is deprecated; use '_BitInt' instead">, InGroup<DeprecatedType>;`.
  **L1780 CN**: 添加一条独立语句或声明：`"'_ExtInt' is deprecated; use '_BitInt' instead">, InGroup<DeprecatedType>;`。
- **L1781 EN**: Declares TableGen def record `ext_bit_int`.
  **L1781 CN**: 声明 TableGen def 记录 `ext_bit_int`。
- **L1782 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'_BitInt' in %select{C17 and earlier|C++}0 is a Clang extension">,`.
  **L1782 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'_BitInt' in %select{C17 and earlier|C++}0 is a Clang extension">,`。
- **L1783 EN**: Adds a standalone statement or declaration: `InGroup<BitIntExtension>;`.
  **L1783 CN**: 添加一条独立语句或声明：`InGroup<BitIntExtension>;`。
- **L1784 EN**: Continues the surrounding expression or declaration: `} // end of Parse Issue category.`.
  **L1784 CN**: 继续构造周围的表达式或声明：`} // end of Parse Issue category.`。
- **L1785 EN**: Blank line separating nearby declarations or logic blocks.
  **L1785 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1786 EN**: Assigns a TableGen property that affects following records or inherited fields: `let CategoryName = "Modules Issue" in {`.
  **L1786 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let CategoryName = "Modules Issue" in {`。
- **L1787 EN**: Declares TableGen def record `err_unexpected_module_or_import_decl`.
  **L1787 CN**: 声明 TableGen def 记录 `err_unexpected_module_or_import_decl`。
- **L1788 EN**: Adds a standalone statement or declaration: `"%select{module|import}0 declaration can only appear at the top level">;`.
  **L1788 CN**: 添加一条独立语句或声明：`"%select{module|import}0 declaration can only appear at the top level">;`。
- **L1789 EN**: Declares TableGen def record `err_attribute_not_module_attr`.
  **L1789 CN**: 声明 TableGen def 记录 `err_attribute_not_module_attr`。
- **L1790 EN**: Adds a standalone statement or declaration: `"%0 attribute cannot be applied to a module">;`.
  **L1790 CN**: 添加一条独立语句或声明：`"%0 attribute cannot be applied to a module">;`。
- **L1791 EN**: Declares TableGen def record `err_keyword_not_module_attr`.
  **L1791 CN**: 声明 TableGen def 记录 `err_keyword_not_module_attr`。
- **L1792 EN**: Adds a standalone statement or declaration: `"%0 cannot be applied to a module">;`.
  **L1792 CN**: 添加一条独立语句或声明：`"%0 cannot be applied to a module">;`。
- **L1793 EN**: Declares TableGen def record `err_attribute_not_import_attr`.
  **L1793 CN**: 声明 TableGen def 记录 `err_attribute_not_import_attr`。
- **L1794 EN**: Adds a standalone statement or declaration: `"%0 attribute cannot be applied to a module import">;`.
  **L1794 CN**: 添加一条独立语句或声明：`"%0 attribute cannot be applied to a module import">;`。
- **L1795 EN**: Declares TableGen def record `err_keyword_not_import_attr`.
  **L1795 CN**: 声明 TableGen def 记录 `err_keyword_not_import_attr`。
- **L1796 EN**: Adds a standalone statement or declaration: `"%0 cannot be applied to a module import">;`.
  **L1796 CN**: 添加一条独立语句或声明：`"%0 cannot be applied to a module import">;`。
- **L1797 EN**: Declares TableGen def record `err_module_expected_semi`.
  **L1797 CN**: 声明 TableGen def 记录 `err_module_expected_semi`。
- **L1798 EN**: Adds a standalone statement or declaration: `"expected ';' after module name">;`.
  **L1798 CN**: 添加一条独立语句或声明：`"expected ';' after module name">;`。
- **L1799 EN**: Declares TableGen def record `err_expected_semi_after_module_or_import`.
  **L1799 CN**: 声明 TableGen def 记录 `err_expected_semi_after_module_or_import`。
- **L1800 EN**: Adds a standalone statement or declaration: `: Error<"%0 directive must end with a ';'">;`.
  **L1800 CN**: 添加一条独立语句或声明：`: Error<"%0 directive must end with a ';'">;`。

### Lines 1801-1824

````tablegen
def note_module_declared_here : Note<
  "%select{module|import}0 directive defined here">;
def err_global_module_introducer_not_at_start : Error<
  "'module;' introducing a global module fragment can appear only "
  "at the start of the translation unit">;
def err_module_fragment_exported : Error<
  "%select{global|private}0 module fragment cannot be exported">;
def err_private_module_fragment_expected_semi : Error<
  "expected ';' after private module fragment declaration">;
def err_missing_before_module_end : Error<"expected %0 at end of module">;
def err_unsupported_module_partition : Error<
  "module partitions are only supported for C++20 onwards">;
def err_import_not_allowed_here : Error<
  "imports must immediately follow the module declaration">;
def err_partition_import_outside_module : Error<
  "module partition imports must be within a module purview">;
def err_import_in_wrong_fragment : Error<
  "module%select{| partition}0 imports cannot be in the %select{global|private}1 module fragment">;

def err_export_empty : Error<"export declaration cannot be empty">;
}

let CategoryName = "Generics Issue" in {

````
- **L1801 EN**: Declares TableGen def record `note_module_declared_here`.
  **L1801 CN**: 声明 TableGen def 记录 `note_module_declared_here`。
- **L1802 EN**: Adds a standalone statement or declaration: `"%select{module|import}0 directive defined here">;`.
  **L1802 CN**: 添加一条独立语句或声明：`"%select{module|import}0 directive defined here">;`。
- **L1803 EN**: Declares TableGen def record `err_global_module_introducer_not_at_start`.
  **L1803 CN**: 声明 TableGen def 记录 `err_global_module_introducer_not_at_start`。
- **L1804 EN**: Continues the surrounding expression or declaration: `"'module;' introducing a global module fragment can appear only "`.
  **L1804 CN**: 继续构造周围的表达式或声明：`"'module;' introducing a global module fragment can appear only "`。
- **L1805 EN**: Adds a standalone statement or declaration: `"at the start of the translation unit">;`.
  **L1805 CN**: 添加一条独立语句或声明：`"at the start of the translation unit">;`。
- **L1806 EN**: Declares TableGen def record `err_module_fragment_exported`.
  **L1806 CN**: 声明 TableGen def 记录 `err_module_fragment_exported`。
- **L1807 EN**: Adds a standalone statement or declaration: `"%select{global|private}0 module fragment cannot be exported">;`.
  **L1807 CN**: 添加一条独立语句或声明：`"%select{global|private}0 module fragment cannot be exported">;`。
- **L1808 EN**: Declares TableGen def record `err_private_module_fragment_expected_semi`.
  **L1808 CN**: 声明 TableGen def 记录 `err_private_module_fragment_expected_semi`。
- **L1809 EN**: Adds a standalone statement or declaration: `"expected ';' after private module fragment declaration">;`.
  **L1809 CN**: 添加一条独立语句或声明：`"expected ';' after private module fragment declaration">;`。
- **L1810 EN**: Declares TableGen def record `err_missing_before_module_end`.
  **L1810 CN**: 声明 TableGen def 记录 `err_missing_before_module_end`。
- **L1811 EN**: Declares TableGen def record `err_unsupported_module_partition`.
  **L1811 CN**: 声明 TableGen def 记录 `err_unsupported_module_partition`。
- **L1812 EN**: Adds a standalone statement or declaration: `"module partitions are only supported for C++20 onwards">;`.
  **L1812 CN**: 添加一条独立语句或声明：`"module partitions are only supported for C++20 onwards">;`。
- **L1813 EN**: Declares TableGen def record `err_import_not_allowed_here`.
  **L1813 CN**: 声明 TableGen def 记录 `err_import_not_allowed_here`。
- **L1814 EN**: Adds a standalone statement or declaration: `"imports must immediately follow the module declaration">;`.
  **L1814 CN**: 添加一条独立语句或声明：`"imports must immediately follow the module declaration">;`。
- **L1815 EN**: Declares TableGen def record `err_partition_import_outside_module`.
  **L1815 CN**: 声明 TableGen def 记录 `err_partition_import_outside_module`。
- **L1816 EN**: Adds a standalone statement or declaration: `"module partition imports must be within a module purview">;`.
  **L1816 CN**: 添加一条独立语句或声明：`"module partition imports must be within a module purview">;`。
- **L1817 EN**: Declares TableGen def record `err_import_in_wrong_fragment`.
  **L1817 CN**: 声明 TableGen def 记录 `err_import_in_wrong_fragment`。
- **L1818 EN**: Adds a standalone statement or declaration: `"module%select{| partition}0 imports cannot be in the %select{global|private}1 module fragment">;`.
  **L1818 CN**: 添加一条独立语句或声明：`"module%select{| partition}0 imports cannot be in the %select{global|private}1 module fragment">;`。
- **L1819 EN**: Blank line separating nearby declarations or logic blocks.
  **L1819 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1820 EN**: Declares TableGen def record `err_export_empty`.
  **L1820 CN**: 声明 TableGen def 记录 `err_export_empty`。
- **L1821 EN**: Closes the current lexical scope or compound statement.
  **L1821 CN**: 结束当前词法作用域或复合语句块。
- **L1822 EN**: Blank line separating nearby declarations or logic blocks.
  **L1822 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1823 EN**: Assigns a TableGen property that affects following records or inherited fields: `let CategoryName = "Generics Issue" in {`.
  **L1823 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let CategoryName = "Generics Issue" in {`。
- **L1824 EN**: Blank line separating nearby declarations or logic blocks.
  **L1824 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1825-1848

````tablegen
def err_objc_expected_type_parameter : Error<
  "expected type parameter name">;

def err_objc_parameterized_implementation : Error<
  "@implementation cannot have type parameters">;

def err_objc_type_args_after_protocols : Error<
  "protocol qualifiers must precede type arguments">;

def note_meant_to_use_typename : Note<
  "did you mean to use 'typename'?">;
}

let CategoryName = "Coroutines Issue" in {
def err_for_co_await_not_range_for : Error<
  "'co_await' modifier can only be applied to range-based for loop">;
def warn_deprecated_for_co_await : Warning<
  "'for co_await' belongs to CoroutineTS instead of C++20, which is deprecated">,
  InGroup<DeprecatedCoroutine>;
}

let CategoryName = "Concepts Issue" in {
def err_concept_definition_not_identifier : Error<
  "name defined in concept definition must be an identifier">;
````
- **L1825 EN**: Declares TableGen def record `err_objc_expected_type_parameter`.
  **L1825 CN**: 声明 TableGen def 记录 `err_objc_expected_type_parameter`。
- **L1826 EN**: Adds a standalone statement or declaration: `"expected type parameter name">;`.
  **L1826 CN**: 添加一条独立语句或声明：`"expected type parameter name">;`。
- **L1827 EN**: Blank line separating nearby declarations or logic blocks.
  **L1827 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1828 EN**: Declares TableGen def record `err_objc_parameterized_implementation`.
  **L1828 CN**: 声明 TableGen def 记录 `err_objc_parameterized_implementation`。
- **L1829 EN**: Adds a standalone statement or declaration: `"@implementation cannot have type parameters">;`.
  **L1829 CN**: 添加一条独立语句或声明：`"@implementation cannot have type parameters">;`。
- **L1830 EN**: Blank line separating nearby declarations or logic blocks.
  **L1830 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1831 EN**: Declares TableGen def record `err_objc_type_args_after_protocols`.
  **L1831 CN**: 声明 TableGen def 记录 `err_objc_type_args_after_protocols`。
- **L1832 EN**: Adds a standalone statement or declaration: `"protocol qualifiers must precede type arguments">;`.
  **L1832 CN**: 添加一条独立语句或声明：`"protocol qualifiers must precede type arguments">;`。
- **L1833 EN**: Blank line separating nearby declarations or logic blocks.
  **L1833 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1834 EN**: Declares TableGen def record `note_meant_to_use_typename`.
  **L1834 CN**: 声明 TableGen def 记录 `note_meant_to_use_typename`。
- **L1835 EN**: Adds a standalone statement or declaration: `"did you mean to use 'typename'?">;`.
  **L1835 CN**: 添加一条独立语句或声明：`"did you mean to use 'typename'?">;`。
- **L1836 EN**: Closes the current lexical scope or compound statement.
  **L1836 CN**: 结束当前词法作用域或复合语句块。
- **L1837 EN**: Blank line separating nearby declarations or logic blocks.
  **L1837 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1838 EN**: Assigns a TableGen property that affects following records or inherited fields: `let CategoryName = "Coroutines Issue" in {`.
  **L1838 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let CategoryName = "Coroutines Issue" in {`。
- **L1839 EN**: Declares TableGen def record `err_for_co_await_not_range_for`.
  **L1839 CN**: 声明 TableGen def 记录 `err_for_co_await_not_range_for`。
- **L1840 EN**: Adds a standalone statement or declaration: `"'co_await' modifier can only be applied to range-based for loop">;`.
  **L1840 CN**: 添加一条独立语句或声明：`"'co_await' modifier can only be applied to range-based for loop">;`。
- **L1841 EN**: Declares TableGen def record `warn_deprecated_for_co_await`.
  **L1841 CN**: 声明 TableGen def 记录 `warn_deprecated_for_co_await`。
- **L1842 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'for co_await' belongs to CoroutineTS instead of C++20, which is deprecated">,`.
  **L1842 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'for co_await' belongs to CoroutineTS instead of C++20, which is deprecated">,`。
- **L1843 EN**: Adds a standalone statement or declaration: `InGroup<DeprecatedCoroutine>;`.
  **L1843 CN**: 添加一条独立语句或声明：`InGroup<DeprecatedCoroutine>;`。
- **L1844 EN**: Closes the current lexical scope or compound statement.
  **L1844 CN**: 结束当前词法作用域或复合语句块。
- **L1845 EN**: Blank line separating nearby declarations or logic blocks.
  **L1845 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1846 EN**: Assigns a TableGen property that affects following records or inherited fields: `let CategoryName = "Concepts Issue" in {`.
  **L1846 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let CategoryName = "Concepts Issue" in {`。
- **L1847 EN**: Declares TableGen def record `err_concept_definition_not_identifier`.
  **L1847 CN**: 声明 TableGen def 记录 `err_concept_definition_not_identifier`。
- **L1848 EN**: Adds a standalone statement or declaration: `"name defined in concept definition must be an identifier">;`.
  **L1848 CN**: 添加一条独立语句或声明：`"name defined in concept definition must be an identifier">;`。

### Lines 1849-1872

````tablegen
def err_concept_legacy_bool_keyword : Error<
  "ISO C++ does not permit the 'bool' keyword after 'concept'">;
def err_placeholder_expected_auto_or_decltype_auto : Error<
  "expected 'auto' or 'decltype(auto)' after concept name">;
}

let CategoryName = "Reflection Diagnostics" in {
def err_cannot_reflect_operand : Error<
  "unknown or unimplemented reflectable entity">;
}

def warn_max_tokens : Warning<
  "the number of preprocessor source tokens (%0) exceeds this token limit (%1)">,
  InGroup<MaxTokens>, DefaultIgnore;

def warn_max_tokens_total : Warning<
  "the total number of preprocessor source tokens (%0) exceeds the token limit (%1)">,
  InGroup<MaxTokens>, DefaultIgnore;

def note_max_tokens_total_override : Note<"total token limit set here">;

// HLSL Parser Diagnostics

def err_expected_semantic_identifier : Error<
````
- **L1849 EN**: Declares TableGen def record `err_concept_legacy_bool_keyword`.
  **L1849 CN**: 声明 TableGen def 记录 `err_concept_legacy_bool_keyword`。
- **L1850 EN**: Adds a standalone statement or declaration: `"ISO C++ does not permit the 'bool' keyword after 'concept'">;`.
  **L1850 CN**: 添加一条独立语句或声明：`"ISO C++ does not permit the 'bool' keyword after 'concept'">;`。
- **L1851 EN**: Declares TableGen def record `err_placeholder_expected_auto_or_decltype_auto`.
  **L1851 CN**: 声明 TableGen def 记录 `err_placeholder_expected_auto_or_decltype_auto`。
- **L1852 EN**: Executes a call or declaration centered on `'decltype`.
  **L1852 CN**: 执行以 `'decltype` 为核心的调用或声明。
- **L1853 EN**: Closes the current lexical scope or compound statement.
  **L1853 CN**: 结束当前词法作用域或复合语句块。
- **L1854 EN**: Blank line separating nearby declarations or logic blocks.
  **L1854 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1855 EN**: Assigns a TableGen property that affects following records or inherited fields: `let CategoryName = "Reflection Diagnostics" in {`.
  **L1855 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let CategoryName = "Reflection Diagnostics" in {`。
- **L1856 EN**: Declares TableGen def record `err_cannot_reflect_operand`.
  **L1856 CN**: 声明 TableGen def 记录 `err_cannot_reflect_operand`。
- **L1857 EN**: Adds a standalone statement or declaration: `"unknown or unimplemented reflectable entity">;`.
  **L1857 CN**: 添加一条独立语句或声明：`"unknown or unimplemented reflectable entity">;`。
- **L1858 EN**: Closes the current lexical scope or compound statement.
  **L1858 CN**: 结束当前词法作用域或复合语句块。
- **L1859 EN**: Blank line separating nearby declarations or logic blocks.
  **L1859 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1860 EN**: Declares TableGen def record `warn_max_tokens`.
  **L1860 CN**: 声明 TableGen def 记录 `warn_max_tokens`。
- **L1861 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"the number of preprocessor source tokens (%0) exceeds this token limit (%1)">,`.
  **L1861 CN**: 继续一个多行参数列表、初始化器或聚合项：`"the number of preprocessor source tokens (%0) exceeds this token limit (%1)">,`。
- **L1862 EN**: Adds a standalone statement or declaration: `InGroup<MaxTokens>, DefaultIgnore;`.
  **L1862 CN**: 添加一条独立语句或声明：`InGroup<MaxTokens>, DefaultIgnore;`。
- **L1863 EN**: Blank line separating nearby declarations or logic blocks.
  **L1863 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1864 EN**: Declares TableGen def record `warn_max_tokens_total`.
  **L1864 CN**: 声明 TableGen def 记录 `warn_max_tokens_total`。
- **L1865 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"the total number of preprocessor source tokens (%0) exceeds the token limit (%1)">,`.
  **L1865 CN**: 继续一个多行参数列表、初始化器或聚合项：`"the total number of preprocessor source tokens (%0) exceeds the token limit (%1)">,`。
- **L1866 EN**: Adds a standalone statement or declaration: `InGroup<MaxTokens>, DefaultIgnore;`.
  **L1866 CN**: 添加一条独立语句或声明：`InGroup<MaxTokens>, DefaultIgnore;`。
- **L1867 EN**: Blank line separating nearby declarations or logic blocks.
  **L1867 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1868 EN**: Declares TableGen def record `note_max_tokens_total_override`.
  **L1868 CN**: 声明 TableGen def 记录 `note_max_tokens_total_override`。
- **L1869 EN**: Blank line separating nearby declarations or logic blocks.
  **L1869 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1870 EN**: Comment explains nearby logic, constraints, or intent: `HLSL Parser Diagnostics`.
  **L1870 CN**: 注释解释附近代码的逻辑、约束或设计意图：`HLSL Parser Diagnostics`。
- **L1871 EN**: Blank line separating nearby declarations or logic blocks.
  **L1871 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1872 EN**: Declares TableGen def record `err_expected_semantic_identifier`.
  **L1872 CN**: 声明 TableGen def 记录 `err_expected_semantic_identifier`。

### Lines 1873-1896

````tablegen
  "expected HLSL Semantic identifier">;
def err_invalid_declaration_in_hlsl_buffer
    : Error<"invalid declaration inside %select{tbuffer|cbuffer}0">;
def err_hlsl_separate_attr_arg_and_number : Error<"wrong argument format for hlsl attribute, use %0 instead">;
def ext_hlsl_access_specifiers : ExtWarn<
  "access specifiers are a clang HLSL extension">,
  InGroup<HLSLExtension>;
def err_hlsl_unsupported_component : Error<"invalid component '%0' used; expected 'x', 'y', 'z', or 'w'">;
def err_hlsl_packoffset_invalid_reg : Error<"invalid resource class specifier '%0' for packoffset, expected 'c'">;
def err_hlsl_virtual_function
    : Error<"virtual functions are unsupported in HLSL">;
def err_hlsl_virtual_inheritance
    : Error<"virtual inheritance is unsupported in HLSL">;

// HLSL Root Signature Parser Diagnostics
def err_hlsl_invalid_token : Error<"invalid %select{parameter|value}0 of %1">;
def err_hlsl_rootsig_repeat_param : Error<"specified the same parameter '%0' multiple times">;
def err_hlsl_rootsig_missing_param : Error<"did not specify mandatory parameter '%0'">;
def err_hlsl_number_literal_overflow : Error<
  "%select{integer|float}0 literal is too large to be represented as a "
  "%select{32-bit %select{signed|}1 integer|float}0 type">;
def err_hlsl_number_literal_underflow : Error<
  "float literal has a magnitude that is too small to be represented as a float type">;
def err_hlsl_rootsig_non_zero_flag : Error<"flag value is neither a literal 0 nor a named value">;
````
- **L1873 EN**: Adds a standalone statement or declaration: `"expected HLSL Semantic identifier">;`.
  **L1873 CN**: 添加一条独立语句或声明：`"expected HLSL Semantic identifier">;`。
- **L1874 EN**: Declares TableGen def record `err_invalid_declaration_in_hlsl_buffer`.
  **L1874 CN**: 声明 TableGen def 记录 `err_invalid_declaration_in_hlsl_buffer`。
- **L1875 EN**: Adds a standalone statement or declaration: `: Error<"invalid declaration inside %select{tbuffer|cbuffer}0">;`.
  **L1875 CN**: 添加一条独立语句或声明：`: Error<"invalid declaration inside %select{tbuffer|cbuffer}0">;`。
- **L1876 EN**: Declares TableGen def record `err_hlsl_separate_attr_arg_and_number`.
  **L1876 CN**: 声明 TableGen def 记录 `err_hlsl_separate_attr_arg_and_number`。
- **L1877 EN**: Declares TableGen def record `ext_hlsl_access_specifiers`.
  **L1877 CN**: 声明 TableGen def 记录 `ext_hlsl_access_specifiers`。
- **L1878 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"access specifiers are a clang HLSL extension">,`.
  **L1878 CN**: 继续一个多行参数列表、初始化器或聚合项：`"access specifiers are a clang HLSL extension">,`。
- **L1879 EN**: Adds a standalone statement or declaration: `InGroup<HLSLExtension>;`.
  **L1879 CN**: 添加一条独立语句或声明：`InGroup<HLSLExtension>;`。
- **L1880 EN**: Declares TableGen def record `err_hlsl_unsupported_component`.
  **L1880 CN**: 声明 TableGen def 记录 `err_hlsl_unsupported_component`。
- **L1881 EN**: Declares TableGen def record `err_hlsl_packoffset_invalid_reg`.
  **L1881 CN**: 声明 TableGen def 记录 `err_hlsl_packoffset_invalid_reg`。
- **L1882 EN**: Declares TableGen def record `err_hlsl_virtual_function`.
  **L1882 CN**: 声明 TableGen def 记录 `err_hlsl_virtual_function`。
- **L1883 EN**: Adds a standalone statement or declaration: `: Error<"virtual functions are unsupported in HLSL">;`.
  **L1883 CN**: 添加一条独立语句或声明：`: Error<"virtual functions are unsupported in HLSL">;`。
- **L1884 EN**: Declares TableGen def record `err_hlsl_virtual_inheritance`.
  **L1884 CN**: 声明 TableGen def 记录 `err_hlsl_virtual_inheritance`。
- **L1885 EN**: Adds a standalone statement or declaration: `: Error<"virtual inheritance is unsupported in HLSL">;`.
  **L1885 CN**: 添加一条独立语句或声明：`: Error<"virtual inheritance is unsupported in HLSL">;`。
- **L1886 EN**: Blank line separating nearby declarations or logic blocks.
  **L1886 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1887 EN**: Comment explains nearby logic, constraints, or intent: `HLSL Root Signature Parser Diagnostics`.
  **L1887 CN**: 注释解释附近代码的逻辑、约束或设计意图：`HLSL Root Signature Parser Diagnostics`。
- **L1888 EN**: Declares TableGen def record `err_hlsl_invalid_token`.
  **L1888 CN**: 声明 TableGen def 记录 `err_hlsl_invalid_token`。
- **L1889 EN**: Declares TableGen def record `err_hlsl_rootsig_repeat_param`.
  **L1889 CN**: 声明 TableGen def 记录 `err_hlsl_rootsig_repeat_param`。
- **L1890 EN**: Declares TableGen def record `err_hlsl_rootsig_missing_param`.
  **L1890 CN**: 声明 TableGen def 记录 `err_hlsl_rootsig_missing_param`。
- **L1891 EN**: Declares TableGen def record `err_hlsl_number_literal_overflow`.
  **L1891 CN**: 声明 TableGen def 记录 `err_hlsl_number_literal_overflow`。
- **L1892 EN**: Continues the surrounding expression or declaration: `"%select{integer|float}0 literal is too large to be represented as a "`.
  **L1892 CN**: 继续构造周围的表达式或声明：`"%select{integer|float}0 literal is too large to be represented as a "`。
- **L1893 EN**: Adds a standalone statement or declaration: `"%select{32-bit %select{signed|}1 integer|float}0 type">;`.
  **L1893 CN**: 添加一条独立语句或声明：`"%select{32-bit %select{signed|}1 integer|float}0 type">;`。
- **L1894 EN**: Declares TableGen def record `err_hlsl_number_literal_underflow`.
  **L1894 CN**: 声明 TableGen def 记录 `err_hlsl_number_literal_underflow`。
- **L1895 EN**: Adds a standalone statement or declaration: `"float literal has a magnitude that is too small to be represented as a float type">;`.
  **L1895 CN**: 添加一条独立语句或声明：`"float literal has a magnitude that is too small to be represented as a float type">;`。
- **L1896 EN**: Declares TableGen def record `err_hlsl_rootsig_non_zero_flag`.
  **L1896 CN**: 声明 TableGen def 记录 `err_hlsl_rootsig_non_zero_flag`。

### Lines 1897-1898

````tablegen

} // end of Parser diagnostics
````
- **L1897 EN**: Blank line separating nearby declarations or logic blocks.
  **L1897 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1898 EN**: Continues the surrounding expression or declaration: `} // end of Parser diagnostics`.
  **L1898 CN**: 继续构造周围的表达式或声明：`} // end of Parser diagnostics`。

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
- **Diagnostics tables / 诊断表**
  - **EN**: Models diagnostic identifiers, categories, and message metadata consumed by Clang diagnostics.
  - **CN**: 建模 Clang 诊断系统使用的诊断标识、类别与消息元数据。
- **Builtin descriptions / Builtin 描述**
  - **EN**: Describes compiler builtins, their signatures, properties, or target-specific availability.
  - **CN**: 描述编译器 builtin 的签名、属性或目标相关可用性。
- **Attribute metadata / 属性元数据**
  - **EN**: Captures attribute spellings, subjects, semantic flags, and generated helper data.
  - **CN**: 刻画属性的拼写、适用对象、语义标志与生成的辅助数据。
- **ABI contracts / ABI 契约**
  - **EN**: Records ABI-relevant enums and conventions used to represent constructors, destructors, or calling details.
  - **CN**: 记录用于表示构造函数、析构函数或调用细节的 ABI 相关枚举与约定。
- **Type-system support / 类型系统支撑**
  - **EN**: Provides metadata or helpers used to model Clang types and related dispatch logic.
  - **CN**: 提供建模 Clang 类型及相关分发逻辑所需的元数据或辅助工具。
- **Indexing and tooling / 索引与工具支持**
  - **EN**: Exposes APIs used by source indexing, diagnostics retrieval, and IDE tooling.
  - **CN**: 暴露源码索引、诊断提取与 IDE 工具使用的 API。

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: None detected. / 未检测到。
- **Macros / 宏**: No prominent macros detected. / 未检测到明显宏。
- **Types / 类型**: `specifier`, `name`, `to`, `extension`, `member`, `list`, `declaration`, `or`, `template`, `already`, `definition`, `method`
- **Functions or callables / 函数或可调用对象**: `brace`, `throw`, `explicit`, `brackets`, `operator`, `available`, `fenv_access`, `comment`, `float_control`, `pragma`, `device_type`, `supported`
- **TableGen records / TableGen 记录**: `enum_fixed_underlying_type`, `err_asm_qualifier_ignored`, `err_global_asm_qualifier_ignored`, `err_asm_empty`, `err_inline_ms_asm_parsing`, `err_msasm_unsupported_arch`, `err_msasm_unable_to_create_target`, `err_gnu_inline_asm_disabled`, `err_asm_duplicate_qual`, `ext_empty_translation_unit`, `warn_cxx98_compat_top_level_semi`, `ext_extra_semi`, `ext_extra_semi_cxx11`, `warn_extra_semi_after_mem_fn_def`, `warn_null_statement`, `warn_misleading_indentation`
- **Namespaces / 命名空间**: No explicit namespaces detected. / 未检测到显式命名空间。
