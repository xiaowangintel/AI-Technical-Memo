# DiagnosticASTKinds.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/include/clang/Basic/DiagnosticASTKinds.td`
- **Repository / 仓库**: `/root/xw/llvm-project`
- **Purpose (EN)**: Declares foundational compiler metadata, diagnostics, target descriptions, or builtin-related definitions for `DiagnosticASTKinds`.
- **Purpose (CN)**: 声明与 `DiagnosticASTKinds` 相关的编译器基础元数据、诊断信息、目标描述或 builtin 定义。
- **Line Count / 行数**: 1101

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````tablegen
//==--- DiagnosticASTKinds.td - libast diagnostics ------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

let Component = "AST" in {

// Constant expression diagnostics. These (and their users) belong in Sema.
def note_expr_divide_by_zero : Note<"division by zero">;
def note_constexpr_invalid_cast : Note<
  "%enum_select<ConstexprInvalidCastKind>{%Reinterpret{reinterpret_cast}|%Dynamic{dynamic_cast}|"
  "%ThisConversionOrReinterpret{%select{this conversion|cast that performs the conversions "
  "of a reinterpret_cast}1}|%CastFrom{cast from %1}}0"
  " is not allowed in a constant expression"
  "%select{| in C++ standards before C++20||}0">;
def note_constexpr_invalid_void_star_cast : Note<
  "cast from %0 is not allowed in a constant expression "
  "%select{in C++ standards before C++2c|because the pointed object "
  "type %2 is not similar to the target type %3}1">;
def note_constexpr_invalid_downcast : Note<
  "cannot cast object of dynamic type %0 to type %1">;
````
- **L1 EN**: Comment explains nearby logic, constraints, or intent: `DiagnosticASTKinds.td - libast diagnostics`.
  **L1 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DiagnosticASTKinds.td - libast diagnostics`。
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
- **L9 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Component = "AST" in {`.
  **L9 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Component = "AST" in {`。
- **L10 EN**: Blank line separating nearby declarations or logic blocks.
  **L10 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L11 EN**: Comment explains nearby logic, constraints, or intent: `Constant expression diagnostics. These (and their users) belong in Sema.`.
  **L11 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Constant expression diagnostics. These (and their users) belong in Sema.`。
- **L12 EN**: Declares TableGen def record `note_expr_divide_by_zero`.
  **L12 CN**: 声明 TableGen def 记录 `note_expr_divide_by_zero`。
- **L13 EN**: Declares TableGen def record `note_constexpr_invalid_cast`.
  **L13 CN**: 声明 TableGen def 记录 `note_constexpr_invalid_cast`。
- **L14 EN**: Continues the surrounding expression or declaration: `"%enum_select<ConstexprInvalidCastKind>{%Reinterpret{reinterpret_cast}|%Dynamic{dynamic_cast}|"`.
  **L14 CN**: 继续构造周围的表达式或声明：`"%enum_select<ConstexprInvalidCastKind>{%Reinterpret{reinterpret_cast}|%Dynamic{dynamic_cast}|"`。
- **L15 EN**: Continues the surrounding expression or declaration: `"%ThisConversionOrReinterpret{%select{this conversion|cast that performs the conversions "`.
  **L15 CN**: 继续构造周围的表达式或声明：`"%ThisConversionOrReinterpret{%select{this conversion|cast that performs the conversions "`。
- **L16 EN**: Continues the surrounding expression or declaration: `"of a reinterpret_cast}1}|%CastFrom{cast from %1}}0"`.
  **L16 CN**: 继续构造周围的表达式或声明：`"of a reinterpret_cast}1}|%CastFrom{cast from %1}}0"`。
- **L17 EN**: Continues the surrounding expression or declaration: `" is not allowed in a constant expression"`.
  **L17 CN**: 继续构造周围的表达式或声明：`" is not allowed in a constant expression"`。
- **L18 EN**: Adds a standalone statement or declaration: `"%select{| in C++ standards before C++20||}0">;`.
  **L18 CN**: 添加一条独立语句或声明：`"%select{| in C++ standards before C++20||}0">;`。
- **L19 EN**: Declares TableGen def record `note_constexpr_invalid_void_star_cast`.
  **L19 CN**: 声明 TableGen def 记录 `note_constexpr_invalid_void_star_cast`。
- **L20 EN**: Continues the surrounding expression or declaration: `"cast from %0 is not allowed in a constant expression "`.
  **L20 CN**: 继续构造周围的表达式或声明：`"cast from %0 is not allowed in a constant expression "`。
- **L21 EN**: Continues the surrounding expression or declaration: `"%select{in C++ standards before C++2c|because the pointed object "`.
  **L21 CN**: 继续构造周围的表达式或声明：`"%select{in C++ standards before C++2c|because the pointed object "`。
- **L22 EN**: Adds a standalone statement or declaration: `"type %2 is not similar to the target type %3}1">;`.
  **L22 CN**: 添加一条独立语句或声明：`"type %2 is not similar to the target type %3}1">;`。
- **L23 EN**: Declares TableGen def record `note_constexpr_invalid_downcast`.
  **L23 CN**: 声明 TableGen def 记录 `note_constexpr_invalid_downcast`。
- **L24 EN**: Adds a standalone statement or declaration: `"cannot cast object of dynamic type %0 to type %1">;`.
  **L24 CN**: 添加一条独立语句或声明：`"cannot cast object of dynamic type %0 to type %1">;`。

### Lines 25-48

````tablegen
def note_constexpr_overflow : Note<
  "value %0 is outside the range of representable values of type %1">;
def note_constexpr_negative_shift : Note<"negative shift count %0">;
def note_constexpr_large_shift : Note<
  "shift count %0 >= width of type %1 (%2 bit%s2)">;
def note_constexpr_lshift_of_negative : Note<"left shift of negative value %0">;
def note_constexpr_lshift_discards : Note<"signed left shift discards bits">;
def note_constexpr_invalid_function : Note<
  "%select{non-constexpr|undefined}0 %select{function|constructor}1 %2 cannot "
  "be used in a constant expression">;
def note_constexpr_invalid_inhctor : Note<
  "constructor inherited from base class %0 cannot be used in a "
  "constant expression; derived class cannot be implicitly initialized">;
def note_constexpr_no_return : Note<
  "control reached end of constexpr function">;
def note_constexpr_virtual_call : Note<
  "cannot evaluate call to virtual function in a constant expression "
  "in C++ standards before C++20">;
def note_constexpr_pure_virtual_call : Note<
  "pure virtual function %q0 called">;
def note_constexpr_polymorphic_unknown_dynamic_type : Note<
  "%select{|||||virtual function called on|dynamic_cast applied to|"
  "typeid applied to|construction of|destruction of}0 object '%1' "
  "whose dynamic type is not constant">;
````
- **L25 EN**: Declares TableGen def record `note_constexpr_overflow`.
  **L25 CN**: 声明 TableGen def 记录 `note_constexpr_overflow`。
- **L26 EN**: Adds a standalone statement or declaration: `"value %0 is outside the range of representable values of type %1">;`.
  **L26 CN**: 添加一条独立语句或声明：`"value %0 is outside the range of representable values of type %1">;`。
- **L27 EN**: Declares TableGen def record `note_constexpr_negative_shift`.
  **L27 CN**: 声明 TableGen def 记录 `note_constexpr_negative_shift`。
- **L28 EN**: Declares TableGen def record `note_constexpr_large_shift`.
  **L28 CN**: 声明 TableGen def 记录 `note_constexpr_large_shift`。
- **L29 EN**: Executes a call or declaration centered on `%1`.
  **L29 CN**: 执行以 `%1` 为核心的调用或声明。
- **L30 EN**: Declares TableGen def record `note_constexpr_lshift_of_negative`.
  **L30 CN**: 声明 TableGen def 记录 `note_constexpr_lshift_of_negative`。
- **L31 EN**: Declares TableGen def record `note_constexpr_lshift_discards`.
  **L31 CN**: 声明 TableGen def 记录 `note_constexpr_lshift_discards`。
- **L32 EN**: Declares TableGen def record `note_constexpr_invalid_function`.
  **L32 CN**: 声明 TableGen def 记录 `note_constexpr_invalid_function`。
- **L33 EN**: Continues the surrounding expression or declaration: `"%select{non-constexpr|undefined}0 %select{function|constructor}1 %2 cannot "`.
  **L33 CN**: 继续构造周围的表达式或声明：`"%select{non-constexpr|undefined}0 %select{function|constructor}1 %2 cannot "`。
- **L34 EN**: Adds a standalone statement or declaration: `"be used in a constant expression">;`.
  **L34 CN**: 添加一条独立语句或声明：`"be used in a constant expression">;`。
- **L35 EN**: Declares TableGen def record `note_constexpr_invalid_inhctor`.
  **L35 CN**: 声明 TableGen def 记录 `note_constexpr_invalid_inhctor`。
- **L36 EN**: Continues the surrounding expression or declaration: `"constructor inherited from base class %0 cannot be used in a "`.
  **L36 CN**: 继续构造周围的表达式或声明：`"constructor inherited from base class %0 cannot be used in a "`。
- **L37 EN**: Adds a standalone statement or declaration: `"constant expression; derived class cannot be implicitly initialized">;`.
  **L37 CN**: 添加一条独立语句或声明：`"constant expression; derived class cannot be implicitly initialized">;`。
- **L38 EN**: Declares TableGen def record `note_constexpr_no_return`.
  **L38 CN**: 声明 TableGen def 记录 `note_constexpr_no_return`。
- **L39 EN**: Adds a standalone statement or declaration: `"control reached end of constexpr function">;`.
  **L39 CN**: 添加一条独立语句或声明：`"control reached end of constexpr function">;`。
- **L40 EN**: Declares TableGen def record `note_constexpr_virtual_call`.
  **L40 CN**: 声明 TableGen def 记录 `note_constexpr_virtual_call`。
- **L41 EN**: Continues the surrounding expression or declaration: `"cannot evaluate call to virtual function in a constant expression "`.
  **L41 CN**: 继续构造周围的表达式或声明：`"cannot evaluate call to virtual function in a constant expression "`。
- **L42 EN**: Adds a standalone statement or declaration: `"in C++ standards before C++20">;`.
  **L42 CN**: 添加一条独立语句或声明：`"in C++ standards before C++20">;`。
- **L43 EN**: Declares TableGen def record `note_constexpr_pure_virtual_call`.
  **L43 CN**: 声明 TableGen def 记录 `note_constexpr_pure_virtual_call`。
- **L44 EN**: Adds a standalone statement or declaration: `"pure virtual function %q0 called">;`.
  **L44 CN**: 添加一条独立语句或声明：`"pure virtual function %q0 called">;`。
- **L45 EN**: Declares TableGen def record `note_constexpr_polymorphic_unknown_dynamic_type`.
  **L45 CN**: 声明 TableGen def 记录 `note_constexpr_polymorphic_unknown_dynamic_type`。
- **L46 EN**: Continues the surrounding expression or declaration: `"%select{|||||virtual function called on|dynamic_cast applied to|"`.
  **L46 CN**: 继续构造周围的表达式或声明：`"%select{|||||virtual function called on|dynamic_cast applied to|"`。
- **L47 EN**: Continues the surrounding expression or declaration: `"typeid applied to|construction of|destruction of}0 object '%1' "`.
  **L47 CN**: 继续构造周围的表达式或声明：`"typeid applied to|construction of|destruction of}0 object '%1' "`。
- **L48 EN**: Adds a standalone statement or declaration: `"whose dynamic type is not constant">;`.
  **L48 CN**: 添加一条独立语句或声明：`"whose dynamic type is not constant">;`。

### Lines 49-72

````tablegen
def note_constexpr_dynamic_cast_to_reference_failed : Note<
  "reference dynamic_cast failed: %select{"
  "static type %1 of operand is a non-public base class of dynamic type %2|"
  "dynamic type %2 of operand does not have a base class of type %3|"
  "%3 is an ambiguous base class of dynamic type %2 of operand|"
  "%3 is a non-public base class of dynamic type %2 of operand}0">;
def note_constexpr_virtual_base : Note<
  "cannot construct object of type %0 with virtual base class "
  "in a constant expression">;
def note_constexpr_nonliteral : Note<
  "non-literal type %0 cannot be used in a constant expression">;
def note_constexpr_non_global : Note<
  "%select{pointer|reference}0 to %select{|subobject of }1"
  "%select{temporary|%3}2 is not a constant expression">;
def note_constexpr_not_static : Note<
  "address of non-static constexpr variable %0 may differ on each invocation "
  "of the enclosing function; add 'static' to give it a constant address">;
def note_constexpr_dynamic_alloc : Note<
  "%select{pointer|reference}0 to %select{|subobject of }1"
  "heap-allocated object is not a constant expression">;
def note_consteval_address_accessible : Note<
  "%select{pointer|reference}0 to a consteval declaration "
  "is not a constant expression">;
def note_constexpr_uninitialized : Note<
````
- **L49 EN**: Declares TableGen def record `note_constexpr_dynamic_cast_to_reference_failed`.
  **L49 CN**: 声明 TableGen def 记录 `note_constexpr_dynamic_cast_to_reference_failed`。
- **L50 EN**: Continues the surrounding expression or declaration: `"reference dynamic_cast failed: %select{"`.
  **L50 CN**: 继续构造周围的表达式或声明：`"reference dynamic_cast failed: %select{"`。
- **L51 EN**: Continues the surrounding expression or declaration: `"static type %1 of operand is a non-public base class of dynamic type %2|"`.
  **L51 CN**: 继续构造周围的表达式或声明：`"static type %1 of operand is a non-public base class of dynamic type %2|"`。
- **L52 EN**: Continues the surrounding expression or declaration: `"dynamic type %2 of operand does not have a base class of type %3|"`.
  **L52 CN**: 继续构造周围的表达式或声明：`"dynamic type %2 of operand does not have a base class of type %3|"`。
- **L53 EN**: Continues the surrounding expression or declaration: `"%3 is an ambiguous base class of dynamic type %2 of operand|"`.
  **L53 CN**: 继续构造周围的表达式或声明：`"%3 is an ambiguous base class of dynamic type %2 of operand|"`。
- **L54 EN**: Adds a standalone statement or declaration: `"%3 is a non-public base class of dynamic type %2 of operand}0">;`.
  **L54 CN**: 添加一条独立语句或声明：`"%3 is a non-public base class of dynamic type %2 of operand}0">;`。
- **L55 EN**: Declares TableGen def record `note_constexpr_virtual_base`.
  **L55 CN**: 声明 TableGen def 记录 `note_constexpr_virtual_base`。
- **L56 EN**: Continues the surrounding expression or declaration: `"cannot construct object of type %0 with virtual base class "`.
  **L56 CN**: 继续构造周围的表达式或声明：`"cannot construct object of type %0 with virtual base class "`。
- **L57 EN**: Adds a standalone statement or declaration: `"in a constant expression">;`.
  **L57 CN**: 添加一条独立语句或声明：`"in a constant expression">;`。
- **L58 EN**: Declares TableGen def record `note_constexpr_nonliteral`.
  **L58 CN**: 声明 TableGen def 记录 `note_constexpr_nonliteral`。
- **L59 EN**: Adds a standalone statement or declaration: `"non-literal type %0 cannot be used in a constant expression">;`.
  **L59 CN**: 添加一条独立语句或声明：`"non-literal type %0 cannot be used in a constant expression">;`。
- **L60 EN**: Declares TableGen def record `note_constexpr_non_global`.
  **L60 CN**: 声明 TableGen def 记录 `note_constexpr_non_global`。
- **L61 EN**: Continues the surrounding expression or declaration: `"%select{pointer|reference}0 to %select{|subobject of }1"`.
  **L61 CN**: 继续构造周围的表达式或声明：`"%select{pointer|reference}0 to %select{|subobject of }1"`。
- **L62 EN**: Adds a standalone statement or declaration: `"%select{temporary|%3}2 is not a constant expression">;`.
  **L62 CN**: 添加一条独立语句或声明：`"%select{temporary|%3}2 is not a constant expression">;`。
- **L63 EN**: Declares TableGen def record `note_constexpr_not_static`.
  **L63 CN**: 声明 TableGen def 记录 `note_constexpr_not_static`。
- **L64 EN**: Continues the surrounding expression or declaration: `"address of non-static constexpr variable %0 may differ on each invocation "`.
  **L64 CN**: 继续构造周围的表达式或声明：`"address of non-static constexpr variable %0 may differ on each invocation "`。
- **L65 EN**: Adds a standalone statement or declaration: `"of the enclosing function; add 'static' to give it a constant address">;`.
  **L65 CN**: 添加一条独立语句或声明：`"of the enclosing function; add 'static' to give it a constant address">;`。
- **L66 EN**: Declares TableGen def record `note_constexpr_dynamic_alloc`.
  **L66 CN**: 声明 TableGen def 记录 `note_constexpr_dynamic_alloc`。
- **L67 EN**: Continues the surrounding expression or declaration: `"%select{pointer|reference}0 to %select{|subobject of }1"`.
  **L67 CN**: 继续构造周围的表达式或声明：`"%select{pointer|reference}0 to %select{|subobject of }1"`。
- **L68 EN**: Adds a standalone statement or declaration: `"heap-allocated object is not a constant expression">;`.
  **L68 CN**: 添加一条独立语句或声明：`"heap-allocated object is not a constant expression">;`。
- **L69 EN**: Declares TableGen def record `note_consteval_address_accessible`.
  **L69 CN**: 声明 TableGen def 记录 `note_consteval_address_accessible`。
- **L70 EN**: Continues the surrounding expression or declaration: `"%select{pointer|reference}0 to a consteval declaration "`.
  **L70 CN**: 继续构造周围的表达式或声明：`"%select{pointer|reference}0 to a consteval declaration "`。
- **L71 EN**: Adds a standalone statement or declaration: `"is not a constant expression">;`.
  **L71 CN**: 添加一条独立语句或声明：`"is not a constant expression">;`。
- **L72 EN**: Declares TableGen def record `note_constexpr_uninitialized`.
  **L72 CN**: 声明 TableGen def 记录 `note_constexpr_uninitialized`。

### Lines 73-96

````tablegen
  "subobject %select{of type |}0%1 is not initialized">;
def note_constexpr_uninitialized_base : Note<
  "constructor of base class %0 is not called">;
def note_constexpr_static_local : Note<
  "control flows through the definition of a %select{static|thread_local}0 variable">;
def note_constexpr_subobject_declared_here : Note<
  "subobject declared here">;
def note_constexpr_array_index : Note<"cannot refer to element %0 of "
  "%select{array of %2 element%plural{1:|:s}2|non-array object}1 "
  "in a constant expression">;
def note_constexpr_float_arithmetic : Note<
  "floating point arithmetic produces %select{an infinity|a NaN}0">;
def note_constexpr_dynamic_rounding : Note<
  "cannot evaluate this expression if rounding mode is dynamic">;
def note_constexpr_float_arithmetic_strict : Note<
  "compile time floating point arithmetic suppressed in strict evaluation modes">;
def note_constexpr_pointer_subtraction_not_same_array : Note<
  "subtracted pointers are not elements of the same array">;
def note_constexpr_pointer_subtraction_zero_size : Note<
  "subtraction of pointers to type %0 of zero size">;
def note_constexpr_pointer_comparison_unspecified : Note<
  "comparison between pointers to unrelated objects '%0' and '%1' has unspecified value">;
def note_constexpr_pointer_arith_unspecified : Note<
  "arithmetic involving unrelated objects '%0' and '%1' has unspecified value">;
````
- **L73 EN**: Adds a standalone statement or declaration: `"subobject %select{of type |}0%1 is not initialized">;`.
  **L73 CN**: 添加一条独立语句或声明：`"subobject %select{of type |}0%1 is not initialized">;`。
- **L74 EN**: Declares TableGen def record `note_constexpr_uninitialized_base`.
  **L74 CN**: 声明 TableGen def 记录 `note_constexpr_uninitialized_base`。
- **L75 EN**: Adds a standalone statement or declaration: `"constructor of base class %0 is not called">;`.
  **L75 CN**: 添加一条独立语句或声明：`"constructor of base class %0 is not called">;`。
- **L76 EN**: Declares TableGen def record `note_constexpr_static_local`.
  **L76 CN**: 声明 TableGen def 记录 `note_constexpr_static_local`。
- **L77 EN**: Adds a standalone statement or declaration: `"control flows through the definition of a %select{static|thread_local}0 variable">;`.
  **L77 CN**: 添加一条独立语句或声明：`"control flows through the definition of a %select{static|thread_local}0 variable">;`。
- **L78 EN**: Declares TableGen def record `note_constexpr_subobject_declared_here`.
  **L78 CN**: 声明 TableGen def 记录 `note_constexpr_subobject_declared_here`。
- **L79 EN**: Adds a standalone statement or declaration: `"subobject declared here">;`.
  **L79 CN**: 添加一条独立语句或声明：`"subobject declared here">;`。
- **L80 EN**: Declares TableGen def record `note_constexpr_array_index`.
  **L80 CN**: 声明 TableGen def 记录 `note_constexpr_array_index`。
- **L81 EN**: Continues the surrounding expression or declaration: `"%select{array of %2 element%plural{1:|:s}2|non-array object}1 "`.
  **L81 CN**: 继续构造周围的表达式或声明：`"%select{array of %2 element%plural{1:|:s}2|non-array object}1 "`。
- **L82 EN**: Adds a standalone statement or declaration: `"in a constant expression">;`.
  **L82 CN**: 添加一条独立语句或声明：`"in a constant expression">;`。
- **L83 EN**: Declares TableGen def record `note_constexpr_float_arithmetic`.
  **L83 CN**: 声明 TableGen def 记录 `note_constexpr_float_arithmetic`。
- **L84 EN**: Adds a standalone statement or declaration: `"floating point arithmetic produces %select{an infinity|a NaN}0">;`.
  **L84 CN**: 添加一条独立语句或声明：`"floating point arithmetic produces %select{an infinity|a NaN}0">;`。
- **L85 EN**: Declares TableGen def record `note_constexpr_dynamic_rounding`.
  **L85 CN**: 声明 TableGen def 记录 `note_constexpr_dynamic_rounding`。
- **L86 EN**: Adds a standalone statement or declaration: `"cannot evaluate this expression if rounding mode is dynamic">;`.
  **L86 CN**: 添加一条独立语句或声明：`"cannot evaluate this expression if rounding mode is dynamic">;`。
- **L87 EN**: Declares TableGen def record `note_constexpr_float_arithmetic_strict`.
  **L87 CN**: 声明 TableGen def 记录 `note_constexpr_float_arithmetic_strict`。
- **L88 EN**: Adds a standalone statement or declaration: `"compile time floating point arithmetic suppressed in strict evaluation modes">;`.
  **L88 CN**: 添加一条独立语句或声明：`"compile time floating point arithmetic suppressed in strict evaluation modes">;`。
- **L89 EN**: Declares TableGen def record `note_constexpr_pointer_subtraction_not_same_array`.
  **L89 CN**: 声明 TableGen def 记录 `note_constexpr_pointer_subtraction_not_same_array`。
- **L90 EN**: Adds a standalone statement or declaration: `"subtracted pointers are not elements of the same array">;`.
  **L90 CN**: 添加一条独立语句或声明：`"subtracted pointers are not elements of the same array">;`。
- **L91 EN**: Declares TableGen def record `note_constexpr_pointer_subtraction_zero_size`.
  **L91 CN**: 声明 TableGen def 记录 `note_constexpr_pointer_subtraction_zero_size`。
- **L92 EN**: Adds a standalone statement or declaration: `"subtraction of pointers to type %0 of zero size">;`.
  **L92 CN**: 添加一条独立语句或声明：`"subtraction of pointers to type %0 of zero size">;`。
- **L93 EN**: Declares TableGen def record `note_constexpr_pointer_comparison_unspecified`.
  **L93 CN**: 声明 TableGen def 记录 `note_constexpr_pointer_comparison_unspecified`。
- **L94 EN**: Adds a standalone statement or declaration: `"comparison between pointers to unrelated objects '%0' and '%1' has unspecified value">;`.
  **L94 CN**: 添加一条独立语句或声明：`"comparison between pointers to unrelated objects '%0' and '%1' has unspecified value">;`。
- **L95 EN**: Declares TableGen def record `note_constexpr_pointer_arith_unspecified`.
  **L95 CN**: 声明 TableGen def 记录 `note_constexpr_pointer_arith_unspecified`。
- **L96 EN**: Adds a standalone statement or declaration: `"arithmetic involving unrelated objects '%0' and '%1' has unspecified value">;`.
  **L96 CN**: 添加一条独立语句或声明：`"arithmetic involving unrelated objects '%0' and '%1' has unspecified value">;`。

### Lines 97-120

````tablegen
def note_constexpr_pointer_constant_comparison : Note<
  "comparison of numeric address '%0' with pointer '%1' can only be performed "
  "at runtime">;
def note_constexpr_literal_comparison : Note<
  "comparison of addresses of potentially overlapping literals has unspecified value">;
def note_constexpr_literal_arith : Note<
  "arithmetic on addresses of potentially overlapping literals has unspecified value">;
def note_constexpr_repeated_literal_eval : Note<
  "repeated evaluation of the same literal expression can produce different objects">;
def note_constexpr_opaque_call_comparison : Note<
  "comparison against opaque constant address '%0' can only be performed at "
  "runtime">;
def note_constexpr_pointer_weak_comparison : Note<
  "comparison against address of weak declaration '%0' can only be performed "
  "at runtime">;
def note_constexpr_mem_pointer_weak_comparison : Note<
  "comparison against pointer to weak member %q0 can only be performed "
  "at runtime">;
def note_constexpr_pointer_comparison_past_end : Note<
  "comparison against pointer '%0' that points past the end of a "
  "complete object has unspecified value">;
def note_constexpr_pointer_comparison_zero_sized : Note<
  "comparison of pointers '%0' and '%1' to unrelated zero-sized objects">;
def note_constexpr_pointer_comparison_base_classes : Note<
````
- **L97 EN**: Declares TableGen def record `note_constexpr_pointer_constant_comparison`.
  **L97 CN**: 声明 TableGen def 记录 `note_constexpr_pointer_constant_comparison`。
- **L98 EN**: Continues the surrounding expression or declaration: `"comparison of numeric address '%0' with pointer '%1' can only be performed "`.
  **L98 CN**: 继续构造周围的表达式或声明：`"comparison of numeric address '%0' with pointer '%1' can only be performed "`。
- **L99 EN**: Adds a standalone statement or declaration: `"at runtime">;`.
  **L99 CN**: 添加一条独立语句或声明：`"at runtime">;`。
- **L100 EN**: Declares TableGen def record `note_constexpr_literal_comparison`.
  **L100 CN**: 声明 TableGen def 记录 `note_constexpr_literal_comparison`。
- **L101 EN**: Adds a standalone statement or declaration: `"comparison of addresses of potentially overlapping literals has unspecified value">;`.
  **L101 CN**: 添加一条独立语句或声明：`"comparison of addresses of potentially overlapping literals has unspecified value">;`。
- **L102 EN**: Declares TableGen def record `note_constexpr_literal_arith`.
  **L102 CN**: 声明 TableGen def 记录 `note_constexpr_literal_arith`。
- **L103 EN**: Adds a standalone statement or declaration: `"arithmetic on addresses of potentially overlapping literals has unspecified value">;`.
  **L103 CN**: 添加一条独立语句或声明：`"arithmetic on addresses of potentially overlapping literals has unspecified value">;`。
- **L104 EN**: Declares TableGen def record `note_constexpr_repeated_literal_eval`.
  **L104 CN**: 声明 TableGen def 记录 `note_constexpr_repeated_literal_eval`。
- **L105 EN**: Adds a standalone statement or declaration: `"repeated evaluation of the same literal expression can produce different objects">;`.
  **L105 CN**: 添加一条独立语句或声明：`"repeated evaluation of the same literal expression can produce different objects">;`。
- **L106 EN**: Declares TableGen def record `note_constexpr_opaque_call_comparison`.
  **L106 CN**: 声明 TableGen def 记录 `note_constexpr_opaque_call_comparison`。
- **L107 EN**: Continues the surrounding expression or declaration: `"comparison against opaque constant address '%0' can only be performed at "`.
  **L107 CN**: 继续构造周围的表达式或声明：`"comparison against opaque constant address '%0' can only be performed at "`。
- **L108 EN**: Adds a standalone statement or declaration: `"runtime">;`.
  **L108 CN**: 添加一条独立语句或声明：`"runtime">;`。
- **L109 EN**: Declares TableGen def record `note_constexpr_pointer_weak_comparison`.
  **L109 CN**: 声明 TableGen def 记录 `note_constexpr_pointer_weak_comparison`。
- **L110 EN**: Continues the surrounding expression or declaration: `"comparison against address of weak declaration '%0' can only be performed "`.
  **L110 CN**: 继续构造周围的表达式或声明：`"comparison against address of weak declaration '%0' can only be performed "`。
- **L111 EN**: Adds a standalone statement or declaration: `"at runtime">;`.
  **L111 CN**: 添加一条独立语句或声明：`"at runtime">;`。
- **L112 EN**: Declares TableGen def record `note_constexpr_mem_pointer_weak_comparison`.
  **L112 CN**: 声明 TableGen def 记录 `note_constexpr_mem_pointer_weak_comparison`。
- **L113 EN**: Continues the surrounding expression or declaration: `"comparison against pointer to weak member %q0 can only be performed "`.
  **L113 CN**: 继续构造周围的表达式或声明：`"comparison against pointer to weak member %q0 can only be performed "`。
- **L114 EN**: Adds a standalone statement or declaration: `"at runtime">;`.
  **L114 CN**: 添加一条独立语句或声明：`"at runtime">;`。
- **L115 EN**: Declares TableGen def record `note_constexpr_pointer_comparison_past_end`.
  **L115 CN**: 声明 TableGen def 记录 `note_constexpr_pointer_comparison_past_end`。
- **L116 EN**: Continues the surrounding expression or declaration: `"comparison against pointer '%0' that points past the end of a "`.
  **L116 CN**: 继续构造周围的表达式或声明：`"comparison against pointer '%0' that points past the end of a "`。
- **L117 EN**: Adds a standalone statement or declaration: `"complete object has unspecified value">;`.
  **L117 CN**: 添加一条独立语句或声明：`"complete object has unspecified value">;`。
- **L118 EN**: Declares TableGen def record `note_constexpr_pointer_comparison_zero_sized`.
  **L118 CN**: 声明 TableGen def 记录 `note_constexpr_pointer_comparison_zero_sized`。
- **L119 EN**: Adds a standalone statement or declaration: `"comparison of pointers '%0' and '%1' to unrelated zero-sized objects">;`.
  **L119 CN**: 添加一条独立语句或声明：`"comparison of pointers '%0' and '%1' to unrelated zero-sized objects">;`。
- **L120 EN**: Declares TableGen def record `note_constexpr_pointer_comparison_base_classes`.
  **L120 CN**: 声明 TableGen def 记录 `note_constexpr_pointer_comparison_base_classes`。

### Lines 121-144

````tablegen
  "comparison of addresses of subobjects of different base classes "
  "has unspecified value">;
def note_constexpr_pointer_comparison_base_field : Note<
  "comparison of address of base class subobject %0 of class %1 to field %2 "
  "has unspecified value">;
def note_constexpr_pointer_comparison_differing_access : Note<
  "comparison of address of fields %0 and %2 of %4 with differing access "
  "specifiers (%1 vs %3) has unspecified value">;
def note_constexpr_compare_virtual_mem_ptr : Note<
  "comparison of pointer to virtual member function %0 has unspecified value">;
def note_constexpr_past_end : Note<
  "dereferenced pointer past the end of %select{|subobject of }0"
  "%select{temporary|%2}1 is not a constant expression">;
def note_constexpr_past_end_subobject : Note<
  "cannot %select{access base class of|access derived class of|access field of|"
  "access array element of|ERROR|"
  "access real component of|access imaginary component of}0 "
  "pointer past the end of object">;
def note_non_null_attribute_failed : Note<
  "null passed to a callee that requires a non-null argument">;
def note_constexpr_null_subobject : Note<
  "cannot %select{access base class of|access derived class of|access field of|"
  "access array element of|perform pointer arithmetic on|"
  "access real component of|"
````
- **L121 EN**: Continues the surrounding expression or declaration: `"comparison of addresses of subobjects of different base classes "`.
  **L121 CN**: 继续构造周围的表达式或声明：`"comparison of addresses of subobjects of different base classes "`。
- **L122 EN**: Adds a standalone statement or declaration: `"has unspecified value">;`.
  **L122 CN**: 添加一条独立语句或声明：`"has unspecified value">;`。
- **L123 EN**: Declares TableGen def record `note_constexpr_pointer_comparison_base_field`.
  **L123 CN**: 声明 TableGen def 记录 `note_constexpr_pointer_comparison_base_field`。
- **L124 EN**: Continues the surrounding expression or declaration: `"comparison of address of base class subobject %0 of class %1 to field %2 "`.
  **L124 CN**: 继续构造周围的表达式或声明：`"comparison of address of base class subobject %0 of class %1 to field %2 "`。
- **L125 EN**: Adds a standalone statement or declaration: `"has unspecified value">;`.
  **L125 CN**: 添加一条独立语句或声明：`"has unspecified value">;`。
- **L126 EN**: Declares TableGen def record `note_constexpr_pointer_comparison_differing_access`.
  **L126 CN**: 声明 TableGen def 记录 `note_constexpr_pointer_comparison_differing_access`。
- **L127 EN**: Continues the surrounding expression or declaration: `"comparison of address of fields %0 and %2 of %4 with differing access "`.
  **L127 CN**: 继续构造周围的表达式或声明：`"comparison of address of fields %0 and %2 of %4 with differing access "`。
- **L128 EN**: Executes a call or declaration centered on `"specifiers`.
  **L128 CN**: 执行以 `"specifiers` 为核心的调用或声明。
- **L129 EN**: Declares TableGen def record `note_constexpr_compare_virtual_mem_ptr`.
  **L129 CN**: 声明 TableGen def 记录 `note_constexpr_compare_virtual_mem_ptr`。
- **L130 EN**: Adds a standalone statement or declaration: `"comparison of pointer to virtual member function %0 has unspecified value">;`.
  **L130 CN**: 添加一条独立语句或声明：`"comparison of pointer to virtual member function %0 has unspecified value">;`。
- **L131 EN**: Declares TableGen def record `note_constexpr_past_end`.
  **L131 CN**: 声明 TableGen def 记录 `note_constexpr_past_end`。
- **L132 EN**: Continues the surrounding expression or declaration: `"dereferenced pointer past the end of %select{|subobject of }0"`.
  **L132 CN**: 继续构造周围的表达式或声明：`"dereferenced pointer past the end of %select{|subobject of }0"`。
- **L133 EN**: Adds a standalone statement or declaration: `"%select{temporary|%2}1 is not a constant expression">;`.
  **L133 CN**: 添加一条独立语句或声明：`"%select{temporary|%2}1 is not a constant expression">;`。
- **L134 EN**: Declares TableGen def record `note_constexpr_past_end_subobject`.
  **L134 CN**: 声明 TableGen def 记录 `note_constexpr_past_end_subobject`。
- **L135 EN**: Continues the surrounding expression or declaration: `"cannot %select{access base class of|access derived class of|access field of|"`.
  **L135 CN**: 继续构造周围的表达式或声明：`"cannot %select{access base class of|access derived class of|access field of|"`。
- **L136 EN**: Continues the surrounding expression or declaration: `"access array element of|ERROR|"`.
  **L136 CN**: 继续构造周围的表达式或声明：`"access array element of|ERROR|"`。
- **L137 EN**: Continues the surrounding expression or declaration: `"access real component of|access imaginary component of}0 "`.
  **L137 CN**: 继续构造周围的表达式或声明：`"access real component of|access imaginary component of}0 "`。
- **L138 EN**: Adds a standalone statement or declaration: `"pointer past the end of object">;`.
  **L138 CN**: 添加一条独立语句或声明：`"pointer past the end of object">;`。
- **L139 EN**: Declares TableGen def record `note_non_null_attribute_failed`.
  **L139 CN**: 声明 TableGen def 记录 `note_non_null_attribute_failed`。
- **L140 EN**: Adds a standalone statement or declaration: `"null passed to a callee that requires a non-null argument">;`.
  **L140 CN**: 添加一条独立语句或声明：`"null passed to a callee that requires a non-null argument">;`。
- **L141 EN**: Declares TableGen def record `note_constexpr_null_subobject`.
  **L141 CN**: 声明 TableGen def 记录 `note_constexpr_null_subobject`。
- **L142 EN**: Continues the surrounding expression or declaration: `"cannot %select{access base class of|access derived class of|access field of|"`.
  **L142 CN**: 继续构造周围的表达式或声明：`"cannot %select{access base class of|access derived class of|access field of|"`。
- **L143 EN**: Continues the surrounding expression or declaration: `"access array element of|perform pointer arithmetic on|"`.
  **L143 CN**: 继续构造周围的表达式或声明：`"access array element of|perform pointer arithmetic on|"`。
- **L144 EN**: Continues the surrounding expression or declaration: `"access real component of|"`.
  **L144 CN**: 继续构造周围的表达式或声明：`"access real component of|"`。

### Lines 145-168

````tablegen
  "access imaginary component of}0 null pointer">;
def note_constexpr_null_callee
    : Note<"%0 evaluates to a null function pointer">;
def note_constexpr_function_param_value_unknown : Note<
  "function parameter %0 with unknown value cannot be used in a constant "
  "expression">;
def note_constexpr_var_init_unknown : Note<
  "initializer of %0 is unknown">;
def note_constexpr_var_init_non_constant : Note<
  "initializer of %0 is not a constant expression">;
def note_constexpr_var_init_weak : Note<
  "initializer of weak variable %0 is not considered constant because "
  "it may be different at runtime">;
def note_constexpr_typeid_polymorphic : Note<
  "typeid applied to expression of polymorphic type %0 is "
  "not allowed in a constant expression in C++ standards before C++20">;
def note_constexpr_temporary_here : Note<"temporary created here">;
def note_constexpr_dynamic_alloc_here : Note<"heap allocation performed here">;
def note_constexpr_conditional_never_const : Note<
  "both arms of conditional operator are unable to produce a "
  "constant expression">;
def note_constexpr_depth_limit_exceeded : Note<
  "constexpr evaluation exceeded maximum depth of %0 calls">;
def note_constexpr_call_limit_exceeded : Note<
````
- **L145 EN**: Adds a standalone statement or declaration: `"access imaginary component of}0 null pointer">;`.
  **L145 CN**: 添加一条独立语句或声明：`"access imaginary component of}0 null pointer">;`。
- **L146 EN**: Declares TableGen def record `note_constexpr_null_callee`.
  **L146 CN**: 声明 TableGen def 记录 `note_constexpr_null_callee`。
- **L147 EN**: Adds a standalone statement or declaration: `: Note<"%0 evaluates to a null function pointer">;`.
  **L147 CN**: 添加一条独立语句或声明：`: Note<"%0 evaluates to a null function pointer">;`。
- **L148 EN**: Declares TableGen def record `note_constexpr_function_param_value_unknown`.
  **L148 CN**: 声明 TableGen def 记录 `note_constexpr_function_param_value_unknown`。
- **L149 EN**: Continues the surrounding expression or declaration: `"function parameter %0 with unknown value cannot be used in a constant "`.
  **L149 CN**: 继续构造周围的表达式或声明：`"function parameter %0 with unknown value cannot be used in a constant "`。
- **L150 EN**: Adds a standalone statement or declaration: `"expression">;`.
  **L150 CN**: 添加一条独立语句或声明：`"expression">;`。
- **L151 EN**: Declares TableGen def record `note_constexpr_var_init_unknown`.
  **L151 CN**: 声明 TableGen def 记录 `note_constexpr_var_init_unknown`。
- **L152 EN**: Adds a standalone statement or declaration: `"initializer of %0 is unknown">;`.
  **L152 CN**: 添加一条独立语句或声明：`"initializer of %0 is unknown">;`。
- **L153 EN**: Declares TableGen def record `note_constexpr_var_init_non_constant`.
  **L153 CN**: 声明 TableGen def 记录 `note_constexpr_var_init_non_constant`。
- **L154 EN**: Adds a standalone statement or declaration: `"initializer of %0 is not a constant expression">;`.
  **L154 CN**: 添加一条独立语句或声明：`"initializer of %0 is not a constant expression">;`。
- **L155 EN**: Declares TableGen def record `note_constexpr_var_init_weak`.
  **L155 CN**: 声明 TableGen def 记录 `note_constexpr_var_init_weak`。
- **L156 EN**: Continues the surrounding expression or declaration: `"initializer of weak variable %0 is not considered constant because "`.
  **L156 CN**: 继续构造周围的表达式或声明：`"initializer of weak variable %0 is not considered constant because "`。
- **L157 EN**: Adds a standalone statement or declaration: `"it may be different at runtime">;`.
  **L157 CN**: 添加一条独立语句或声明：`"it may be different at runtime">;`。
- **L158 EN**: Declares TableGen def record `note_constexpr_typeid_polymorphic`.
  **L158 CN**: 声明 TableGen def 记录 `note_constexpr_typeid_polymorphic`。
- **L159 EN**: Continues the surrounding expression or declaration: `"typeid applied to expression of polymorphic type %0 is "`.
  **L159 CN**: 继续构造周围的表达式或声明：`"typeid applied to expression of polymorphic type %0 is "`。
- **L160 EN**: Adds a standalone statement or declaration: `"not allowed in a constant expression in C++ standards before C++20">;`.
  **L160 CN**: 添加一条独立语句或声明：`"not allowed in a constant expression in C++ standards before C++20">;`。
- **L161 EN**: Declares TableGen def record `note_constexpr_temporary_here`.
  **L161 CN**: 声明 TableGen def 记录 `note_constexpr_temporary_here`。
- **L162 EN**: Declares TableGen def record `note_constexpr_dynamic_alloc_here`.
  **L162 CN**: 声明 TableGen def 记录 `note_constexpr_dynamic_alloc_here`。
- **L163 EN**: Declares TableGen def record `note_constexpr_conditional_never_const`.
  **L163 CN**: 声明 TableGen def 记录 `note_constexpr_conditional_never_const`。
- **L164 EN**: Continues the surrounding expression or declaration: `"both arms of conditional operator are unable to produce a "`.
  **L164 CN**: 继续构造周围的表达式或声明：`"both arms of conditional operator are unable to produce a "`。
- **L165 EN**: Adds a standalone statement or declaration: `"constant expression">;`.
  **L165 CN**: 添加一条独立语句或声明：`"constant expression">;`。
- **L166 EN**: Declares TableGen def record `note_constexpr_depth_limit_exceeded`.
  **L166 CN**: 声明 TableGen def 记录 `note_constexpr_depth_limit_exceeded`。
- **L167 EN**: Adds a standalone statement or declaration: `"constexpr evaluation exceeded maximum depth of %0 calls">;`.
  **L167 CN**: 添加一条独立语句或声明：`"constexpr evaluation exceeded maximum depth of %0 calls">;`。
- **L168 EN**: Declares TableGen def record `note_constexpr_call_limit_exceeded`.
  **L168 CN**: 声明 TableGen def 记录 `note_constexpr_call_limit_exceeded`。

### Lines 169-192

````tablegen
  "constexpr evaluation hit maximum call limit">;
def note_constexpr_step_limit_exceeded : Note<
  "constexpr evaluation hit maximum step limit; possible infinite loop?">;
def note_constexpr_heap_alloc_limit_exceeded : Note<
  "constexpr evaluation hit maximum heap allocation limit">;
def note_constexpr_this : Note<
  "%select{|implicit }0use of 'this' pointer is only allowed within the "
  "evaluation of a call to a 'constexpr' member function">;
def access_kind
    : TextSubstitution<
          "%select{read of|read of|assignment to|increment of|decrement of|"
          "member call on|dynamic_cast of|typeid applied to|construction of|"
          "destruction of|read of|read of}0">;
def access_kind_subobject : TextSubstitution<
  "%select{read of|read of|assignment to|increment of|decrement of|"
  "member call on|dynamic_cast of|typeid applied to|"
  "construction of subobject of|destruction of|read of}0">;
def access_kind_volatile : TextSubstitution<
  "%select{read of|read of|assignment to|increment of|decrement of|"
  "<ERROR>|<ERROR>|<ERROR>|<ERROR>|<ERROR>|<ERROR>}0">;
def note_constexpr_access_uninit : Note<
  "%sub{access_kind_subobject}0 "
  "%select{object outside its lifetime|uninitialized object}1 "
  "is not allowed in a constant expression">;
````
- **L169 EN**: Adds a standalone statement or declaration: `"constexpr evaluation hit maximum call limit">;`.
  **L169 CN**: 添加一条独立语句或声明：`"constexpr evaluation hit maximum call limit">;`。
- **L170 EN**: Declares TableGen def record `note_constexpr_step_limit_exceeded`.
  **L170 CN**: 声明 TableGen def 记录 `note_constexpr_step_limit_exceeded`。
- **L171 EN**: Adds a standalone statement or declaration: `"constexpr evaluation hit maximum step limit; possible infinite loop?">;`.
  **L171 CN**: 添加一条独立语句或声明：`"constexpr evaluation hit maximum step limit; possible infinite loop?">;`。
- **L172 EN**: Declares TableGen def record `note_constexpr_heap_alloc_limit_exceeded`.
  **L172 CN**: 声明 TableGen def 记录 `note_constexpr_heap_alloc_limit_exceeded`。
- **L173 EN**: Adds a standalone statement or declaration: `"constexpr evaluation hit maximum heap allocation limit">;`.
  **L173 CN**: 添加一条独立语句或声明：`"constexpr evaluation hit maximum heap allocation limit">;`。
- **L174 EN**: Declares TableGen def record `note_constexpr_this`.
  **L174 CN**: 声明 TableGen def 记录 `note_constexpr_this`。
- **L175 EN**: Continues the surrounding expression or declaration: `"%select{|implicit }0use of 'this' pointer is only allowed within the "`.
  **L175 CN**: 继续构造周围的表达式或声明：`"%select{|implicit }0use of 'this' pointer is only allowed within the "`。
- **L176 EN**: Adds a standalone statement or declaration: `"evaluation of a call to a 'constexpr' member function">;`.
  **L176 CN**: 添加一条独立语句或声明：`"evaluation of a call to a 'constexpr' member function">;`。
- **L177 EN**: Declares TableGen def record `access_kind`.
  **L177 CN**: 声明 TableGen def 记录 `access_kind`。
- **L178 EN**: Continues the surrounding expression or declaration: `: TextSubstitution<`.
  **L178 CN**: 继续构造周围的表达式或声明：`: TextSubstitution<`。
- **L179 EN**: Continues the surrounding expression or declaration: `"%select{read of|read of|assignment to|increment of|decrement of|"`.
  **L179 CN**: 继续构造周围的表达式或声明：`"%select{read of|read of|assignment to|increment of|decrement of|"`。
- **L180 EN**: Continues the surrounding expression or declaration: `"member call on|dynamic_cast of|typeid applied to|construction of|"`.
  **L180 CN**: 继续构造周围的表达式或声明：`"member call on|dynamic_cast of|typeid applied to|construction of|"`。
- **L181 EN**: Adds a standalone statement or declaration: `"destruction of|read of|read of}0">;`.
  **L181 CN**: 添加一条独立语句或声明：`"destruction of|read of|read of}0">;`。
- **L182 EN**: Declares TableGen def record `access_kind_subobject`.
  **L182 CN**: 声明 TableGen def 记录 `access_kind_subobject`。
- **L183 EN**: Continues the surrounding expression or declaration: `"%select{read of|read of|assignment to|increment of|decrement of|"`.
  **L183 CN**: 继续构造周围的表达式或声明：`"%select{read of|read of|assignment to|increment of|decrement of|"`。
- **L184 EN**: Continues the surrounding expression or declaration: `"member call on|dynamic_cast of|typeid applied to|"`.
  **L184 CN**: 继续构造周围的表达式或声明：`"member call on|dynamic_cast of|typeid applied to|"`。
- **L185 EN**: Adds a standalone statement or declaration: `"construction of subobject of|destruction of|read of}0">;`.
  **L185 CN**: 添加一条独立语句或声明：`"construction of subobject of|destruction of|read of}0">;`。
- **L186 EN**: Declares TableGen def record `access_kind_volatile`.
  **L186 CN**: 声明 TableGen def 记录 `access_kind_volatile`。
- **L187 EN**: Continues the surrounding expression or declaration: `"%select{read of|read of|assignment to|increment of|decrement of|"`.
  **L187 CN**: 继续构造周围的表达式或声明：`"%select{read of|read of|assignment to|increment of|decrement of|"`。
- **L188 EN**: Adds a standalone statement or declaration: `"<ERROR>|<ERROR>|<ERROR>|<ERROR>|<ERROR>|<ERROR>}0">;`.
  **L188 CN**: 添加一条独立语句或声明：`"<ERROR>|<ERROR>|<ERROR>|<ERROR>|<ERROR>|<ERROR>}0">;`。
- **L189 EN**: Declares TableGen def record `note_constexpr_access_uninit`.
  **L189 CN**: 声明 TableGen def 记录 `note_constexpr_access_uninit`。
- **L190 EN**: Continues the surrounding expression or declaration: `"%sub{access_kind_subobject}0 "`.
  **L190 CN**: 继续构造周围的表达式或声明：`"%sub{access_kind_subobject}0 "`。
- **L191 EN**: Continues the surrounding expression or declaration: `"%select{object outside its lifetime|uninitialized object}1 "`.
  **L191 CN**: 继续构造周围的表达式或声明：`"%select{object outside its lifetime|uninitialized object}1 "`。
- **L192 EN**: Adds a standalone statement or declaration: `"is not allowed in a constant expression">;`.
  **L192 CN**: 添加一条独立语句或声明：`"is not allowed in a constant expression">;`。

### Lines 193-216

````tablegen
def note_constexpr_use_uninit_reference : Note<
  "use of reference outside its lifetime "
  "is not allowed in a constant expression">;
def note_constexpr_modify_const_type : Note<
  "modification of object of const-qualified type %0 is not allowed "
  "in a constant expression">;
def note_constexpr_access_volatile_type : Note<
  "%sub{access_kind_volatile}0 "
  "volatile-qualified type %1 is not allowed in a constant expression">;
def note_constexpr_access_volatile_obj : Note<
  "%sub{access_kind_volatile}0 "
  "volatile %select{temporary|object %2|member %2}1 is not allowed in "
  "a constant expression">;
def note_constexpr_volatile_here : Note<
  "volatile %select{temporary created|object declared|member declared}0 here">;
def note_constexpr_access_mutable : Note<
  "%sub{access_kind}0 "
  "mutable member %1 is not allowed in a constant expression">;
def note_constexpr_ltor_non_const_int : Note<
  "read of non-const variable %0 is not allowed in a constant expression">;
def note_constexpr_ltor_non_integral : Note<
  "read of variable %0 of non-integral, non-enumeration type %1 "
  "is not allowed in a constant expression">;
def note_constexpr_ltor_non_constexpr : Note<
````
- **L193 EN**: Declares TableGen def record `note_constexpr_use_uninit_reference`.
  **L193 CN**: 声明 TableGen def 记录 `note_constexpr_use_uninit_reference`。
- **L194 EN**: Continues the surrounding expression or declaration: `"use of reference outside its lifetime "`.
  **L194 CN**: 继续构造周围的表达式或声明：`"use of reference outside its lifetime "`。
- **L195 EN**: Adds a standalone statement or declaration: `"is not allowed in a constant expression">;`.
  **L195 CN**: 添加一条独立语句或声明：`"is not allowed in a constant expression">;`。
- **L196 EN**: Declares TableGen def record `note_constexpr_modify_const_type`.
  **L196 CN**: 声明 TableGen def 记录 `note_constexpr_modify_const_type`。
- **L197 EN**: Continues the surrounding expression or declaration: `"modification of object of const-qualified type %0 is not allowed "`.
  **L197 CN**: 继续构造周围的表达式或声明：`"modification of object of const-qualified type %0 is not allowed "`。
- **L198 EN**: Adds a standalone statement or declaration: `"in a constant expression">;`.
  **L198 CN**: 添加一条独立语句或声明：`"in a constant expression">;`。
- **L199 EN**: Declares TableGen def record `note_constexpr_access_volatile_type`.
  **L199 CN**: 声明 TableGen def 记录 `note_constexpr_access_volatile_type`。
- **L200 EN**: Continues the surrounding expression or declaration: `"%sub{access_kind_volatile}0 "`.
  **L200 CN**: 继续构造周围的表达式或声明：`"%sub{access_kind_volatile}0 "`。
- **L201 EN**: Adds a standalone statement or declaration: `"volatile-qualified type %1 is not allowed in a constant expression">;`.
  **L201 CN**: 添加一条独立语句或声明：`"volatile-qualified type %1 is not allowed in a constant expression">;`。
- **L202 EN**: Declares TableGen def record `note_constexpr_access_volatile_obj`.
  **L202 CN**: 声明 TableGen def 记录 `note_constexpr_access_volatile_obj`。
- **L203 EN**: Continues the surrounding expression or declaration: `"%sub{access_kind_volatile}0 "`.
  **L203 CN**: 继续构造周围的表达式或声明：`"%sub{access_kind_volatile}0 "`。
- **L204 EN**: Continues the surrounding expression or declaration: `"volatile %select{temporary|object %2|member %2}1 is not allowed in "`.
  **L204 CN**: 继续构造周围的表达式或声明：`"volatile %select{temporary|object %2|member %2}1 is not allowed in "`。
- **L205 EN**: Adds a standalone statement or declaration: `"a constant expression">;`.
  **L205 CN**: 添加一条独立语句或声明：`"a constant expression">;`。
- **L206 EN**: Declares TableGen def record `note_constexpr_volatile_here`.
  **L206 CN**: 声明 TableGen def 记录 `note_constexpr_volatile_here`。
- **L207 EN**: Adds a standalone statement or declaration: `"volatile %select{temporary created|object declared|member declared}0 here">;`.
  **L207 CN**: 添加一条独立语句或声明：`"volatile %select{temporary created|object declared|member declared}0 here">;`。
- **L208 EN**: Declares TableGen def record `note_constexpr_access_mutable`.
  **L208 CN**: 声明 TableGen def 记录 `note_constexpr_access_mutable`。
- **L209 EN**: Continues the surrounding expression or declaration: `"%sub{access_kind}0 "`.
  **L209 CN**: 继续构造周围的表达式或声明：`"%sub{access_kind}0 "`。
- **L210 EN**: Adds a standalone statement or declaration: `"mutable member %1 is not allowed in a constant expression">;`.
  **L210 CN**: 添加一条独立语句或声明：`"mutable member %1 is not allowed in a constant expression">;`。
- **L211 EN**: Declares TableGen def record `note_constexpr_ltor_non_const_int`.
  **L211 CN**: 声明 TableGen def 记录 `note_constexpr_ltor_non_const_int`。
- **L212 EN**: Adds a standalone statement or declaration: `"read of non-const variable %0 is not allowed in a constant expression">;`.
  **L212 CN**: 添加一条独立语句或声明：`"read of non-const variable %0 is not allowed in a constant expression">;`。
- **L213 EN**: Declares TableGen def record `note_constexpr_ltor_non_integral`.
  **L213 CN**: 声明 TableGen def 记录 `note_constexpr_ltor_non_integral`。
- **L214 EN**: Continues the surrounding expression or declaration: `"read of variable %0 of non-integral, non-enumeration type %1 "`.
  **L214 CN**: 继续构造周围的表达式或声明：`"read of variable %0 of non-integral, non-enumeration type %1 "`。
- **L215 EN**: Adds a standalone statement or declaration: `"is not allowed in a constant expression">;`.
  **L215 CN**: 添加一条独立语句或声明：`"is not allowed in a constant expression">;`。
- **L216 EN**: Declares TableGen def record `note_constexpr_ltor_non_constexpr`.
  **L216 CN**: 声明 TableGen def 记录 `note_constexpr_ltor_non_constexpr`。

### Lines 217-240

````tablegen
  "read of non-constexpr variable %0 is not allowed in a constant expression">;
def note_constexpr_ltor_incomplete_type : Note<
  "read of incomplete type %0 is not allowed in a constant expression">;
def note_constexpr_access_null : Note<
  "%sub{access_kind}0 "
  "dereferenced null pointer is not allowed in a constant expression">;
def note_constexpr_dereferencing_null
    : Note<"dereferencing a null pointer is not allowed in a constant "
           "expression">;
def note_constexpr_access_past_end : Note<
  "%sub{access_kind}0 dereferenced one-past-the-end pointer "
  "is not allowed in a constant expression">;
def note_constexpr_access_unsized_array : Note<
  "%sub{access_kind}0 element of array without known bound "
  "is not allowed in a constant expression">;
def note_constexpr_access_inactive_union_member : Note<
  "%sub{access_kind_subobject}0 "
  "member %1 of union with %select{active member %3|no active member}2 "
  "is not allowed in a constant expression">;
def note_constexpr_union_member_change_during_init : Note<
  "assignment would change active union member during the initialization of "
  "a different member of the same union">;
def note_constexpr_access_static_temporary : Note<
  "%sub{access_kind}0 temporary is not allowed in a constant expression "
````
- **L217 EN**: Adds a standalone statement or declaration: `"read of non-constexpr variable %0 is not allowed in a constant expression">;`.
  **L217 CN**: 添加一条独立语句或声明：`"read of non-constexpr variable %0 is not allowed in a constant expression">;`。
- **L218 EN**: Declares TableGen def record `note_constexpr_ltor_incomplete_type`.
  **L218 CN**: 声明 TableGen def 记录 `note_constexpr_ltor_incomplete_type`。
- **L219 EN**: Adds a standalone statement or declaration: `"read of incomplete type %0 is not allowed in a constant expression">;`.
  **L219 CN**: 添加一条独立语句或声明：`"read of incomplete type %0 is not allowed in a constant expression">;`。
- **L220 EN**: Declares TableGen def record `note_constexpr_access_null`.
  **L220 CN**: 声明 TableGen def 记录 `note_constexpr_access_null`。
- **L221 EN**: Continues the surrounding expression or declaration: `"%sub{access_kind}0 "`.
  **L221 CN**: 继续构造周围的表达式或声明：`"%sub{access_kind}0 "`。
- **L222 EN**: Adds a standalone statement or declaration: `"dereferenced null pointer is not allowed in a constant expression">;`.
  **L222 CN**: 添加一条独立语句或声明：`"dereferenced null pointer is not allowed in a constant expression">;`。
- **L223 EN**: Declares TableGen def record `note_constexpr_dereferencing_null`.
  **L223 CN**: 声明 TableGen def 记录 `note_constexpr_dereferencing_null`。
- **L224 EN**: Continues the surrounding expression or declaration: `: Note<"dereferencing a null pointer is not allowed in a constant "`.
  **L224 CN**: 继续构造周围的表达式或声明：`: Note<"dereferencing a null pointer is not allowed in a constant "`。
- **L225 EN**: Adds a standalone statement or declaration: `"expression">;`.
  **L225 CN**: 添加一条独立语句或声明：`"expression">;`。
- **L226 EN**: Declares TableGen def record `note_constexpr_access_past_end`.
  **L226 CN**: 声明 TableGen def 记录 `note_constexpr_access_past_end`。
- **L227 EN**: Continues the surrounding expression or declaration: `"%sub{access_kind}0 dereferenced one-past-the-end pointer "`.
  **L227 CN**: 继续构造周围的表达式或声明：`"%sub{access_kind}0 dereferenced one-past-the-end pointer "`。
- **L228 EN**: Adds a standalone statement or declaration: `"is not allowed in a constant expression">;`.
  **L228 CN**: 添加一条独立语句或声明：`"is not allowed in a constant expression">;`。
- **L229 EN**: Declares TableGen def record `note_constexpr_access_unsized_array`.
  **L229 CN**: 声明 TableGen def 记录 `note_constexpr_access_unsized_array`。
- **L230 EN**: Continues the surrounding expression or declaration: `"%sub{access_kind}0 element of array without known bound "`.
  **L230 CN**: 继续构造周围的表达式或声明：`"%sub{access_kind}0 element of array without known bound "`。
- **L231 EN**: Adds a standalone statement or declaration: `"is not allowed in a constant expression">;`.
  **L231 CN**: 添加一条独立语句或声明：`"is not allowed in a constant expression">;`。
- **L232 EN**: Declares TableGen def record `note_constexpr_access_inactive_union_member`.
  **L232 CN**: 声明 TableGen def 记录 `note_constexpr_access_inactive_union_member`。
- **L233 EN**: Continues the surrounding expression or declaration: `"%sub{access_kind_subobject}0 "`.
  **L233 CN**: 继续构造周围的表达式或声明：`"%sub{access_kind_subobject}0 "`。
- **L234 EN**: Continues the surrounding expression or declaration: `"member %1 of union with %select{active member %3|no active member}2 "`.
  **L234 CN**: 继续构造周围的表达式或声明：`"member %1 of union with %select{active member %3|no active member}2 "`。
- **L235 EN**: Adds a standalone statement or declaration: `"is not allowed in a constant expression">;`.
  **L235 CN**: 添加一条独立语句或声明：`"is not allowed in a constant expression">;`。
- **L236 EN**: Declares TableGen def record `note_constexpr_union_member_change_during_init`.
  **L236 CN**: 声明 TableGen def 记录 `note_constexpr_union_member_change_during_init`。
- **L237 EN**: Continues the surrounding expression or declaration: `"assignment would change active union member during the initialization of "`.
  **L237 CN**: 继续构造周围的表达式或声明：`"assignment would change active union member during the initialization of "`。
- **L238 EN**: Adds a standalone statement or declaration: `"a different member of the same union">;`.
  **L238 CN**: 添加一条独立语句或声明：`"a different member of the same union">;`。
- **L239 EN**: Declares TableGen def record `note_constexpr_access_static_temporary`.
  **L239 CN**: 声明 TableGen def 记录 `note_constexpr_access_static_temporary`。
- **L240 EN**: Continues the surrounding expression or declaration: `"%sub{access_kind}0 temporary is not allowed in a constant expression "`.
  **L240 CN**: 继续构造周围的表达式或声明：`"%sub{access_kind}0 temporary is not allowed in a constant expression "`。

### Lines 241-264

````tablegen
  "outside the expression that created the temporary">;
def note_constexpr_access_unreadable_object : Note<
  "%sub{access_kind}0 object '%1' whose value is not known">;
def note_constexpr_access_unknown_variable : Note<
  "%sub{access_kind}0 variable %1 whose value is not known">;
def note_constexpr_access_deleted_object : Note<
  "%sub{access_kind}0 heap allocated object that has been deleted">;
def note_constexpr_modify_global : Note<
  "a constant expression cannot modify an object that is visible outside "
  "that expression">;
def note_constexpr_stmt_expr_unsupported : Note<
  "this use of statement expressions is not supported in a "
  "constant expression">;
def note_constexpr_calls_suppressed : Note<
  "(skipping %0 call%s0 in backtrace; use -fconstexpr-backtrace-limit=0 to "
  "see all)">;
def note_constexpr_call_here : Note<"in call to '%0'">;
def note_constexpr_inherited_ctor_call_here : Note<
  "in implicit initialization for inherited constructor of %0">;
def note_constexpr_baa_insufficient_alignment : Note<
  "%select{alignment of|offset of the aligned pointer from}0 the base pointee "
  "object (%1 %plural{1:byte|:bytes}1) is %select{less than|not a multiple of}0 the "
  "asserted %2 %plural{1:byte|:bytes}2">;
def note_constexpr_baa_value_insufficient_alignment : Note<
````
- **L241 EN**: Adds a standalone statement or declaration: `"outside the expression that created the temporary">;`.
  **L241 CN**: 添加一条独立语句或声明：`"outside the expression that created the temporary">;`。
- **L242 EN**: Declares TableGen def record `note_constexpr_access_unreadable_object`.
  **L242 CN**: 声明 TableGen def 记录 `note_constexpr_access_unreadable_object`。
- **L243 EN**: Adds a standalone statement or declaration: `"%sub{access_kind}0 object '%1' whose value is not known">;`.
  **L243 CN**: 添加一条独立语句或声明：`"%sub{access_kind}0 object '%1' whose value is not known">;`。
- **L244 EN**: Declares TableGen def record `note_constexpr_access_unknown_variable`.
  **L244 CN**: 声明 TableGen def 记录 `note_constexpr_access_unknown_variable`。
- **L245 EN**: Adds a standalone statement or declaration: `"%sub{access_kind}0 variable %1 whose value is not known">;`.
  **L245 CN**: 添加一条独立语句或声明：`"%sub{access_kind}0 variable %1 whose value is not known">;`。
- **L246 EN**: Declares TableGen def record `note_constexpr_access_deleted_object`.
  **L246 CN**: 声明 TableGen def 记录 `note_constexpr_access_deleted_object`。
- **L247 EN**: Adds a standalone statement or declaration: `"%sub{access_kind}0 heap allocated object that has been deleted">;`.
  **L247 CN**: 添加一条独立语句或声明：`"%sub{access_kind}0 heap allocated object that has been deleted">;`。
- **L248 EN**: Declares TableGen def record `note_constexpr_modify_global`.
  **L248 CN**: 声明 TableGen def 记录 `note_constexpr_modify_global`。
- **L249 EN**: Continues the surrounding expression or declaration: `"a constant expression cannot modify an object that is visible outside "`.
  **L249 CN**: 继续构造周围的表达式或声明：`"a constant expression cannot modify an object that is visible outside "`。
- **L250 EN**: Adds a standalone statement or declaration: `"that expression">;`.
  **L250 CN**: 添加一条独立语句或声明：`"that expression">;`。
- **L251 EN**: Declares TableGen def record `note_constexpr_stmt_expr_unsupported`.
  **L251 CN**: 声明 TableGen def 记录 `note_constexpr_stmt_expr_unsupported`。
- **L252 EN**: Continues the surrounding expression or declaration: `"this use of statement expressions is not supported in a "`.
  **L252 CN**: 继续构造周围的表达式或声明：`"this use of statement expressions is not supported in a "`。
- **L253 EN**: Adds a standalone statement or declaration: `"constant expression">;`.
  **L253 CN**: 添加一条独立语句或声明：`"constant expression">;`。
- **L254 EN**: Declares TableGen def record `note_constexpr_calls_suppressed`.
  **L254 CN**: 声明 TableGen def 记录 `note_constexpr_calls_suppressed`。
- **L255 EN**: Continues the surrounding expression or declaration: `"(skipping %0 call%s0 in backtrace; use -fconstexpr-backtrace-limit=0 to "`.
  **L255 CN**: 继续构造周围的表达式或声明：`"(skipping %0 call%s0 in backtrace; use -fconstexpr-backtrace-limit=0 to "`。
- **L256 EN**: Adds a standalone statement or declaration: `"see all)">;`.
  **L256 CN**: 添加一条独立语句或声明：`"see all)">;`。
- **L257 EN**: Declares TableGen def record `note_constexpr_call_here`.
  **L257 CN**: 声明 TableGen def 记录 `note_constexpr_call_here`。
- **L258 EN**: Declares TableGen def record `note_constexpr_inherited_ctor_call_here`.
  **L258 CN**: 声明 TableGen def 记录 `note_constexpr_inherited_ctor_call_here`。
- **L259 EN**: Adds a standalone statement or declaration: `"in implicit initialization for inherited constructor of %0">;`.
  **L259 CN**: 添加一条独立语句或声明：`"in implicit initialization for inherited constructor of %0">;`。
- **L260 EN**: Declares TableGen def record `note_constexpr_baa_insufficient_alignment`.
  **L260 CN**: 声明 TableGen def 记录 `note_constexpr_baa_insufficient_alignment`。
- **L261 EN**: Continues the surrounding expression or declaration: `"%select{alignment of|offset of the aligned pointer from}0 the base pointee "`.
  **L261 CN**: 继续构造周围的表达式或声明：`"%select{alignment of|offset of the aligned pointer from}0 the base pointee "`。
- **L262 EN**: Continues logic associated with callable symbol `object`.
  **L262 CN**: 继续与可调用符号 `object` 相关的逻辑。
- **L263 EN**: Adds a standalone statement or declaration: `"asserted %2 %plural{1:byte|:bytes}2">;`.
  **L263 CN**: 添加一条独立语句或声明：`"asserted %2 %plural{1:byte|:bytes}2">;`。
- **L264 EN**: Declares TableGen def record `note_constexpr_baa_value_insufficient_alignment`.
  **L264 CN**: 声明 TableGen def 记录 `note_constexpr_baa_value_insufficient_alignment`。

### Lines 265-288

````tablegen
  "value of the aligned pointer (%0) is not a multiple of the asserted %1 "
  "%plural{1:byte|:bytes}1">;
def note_constexpr_invalid_alignment : Note<
  "requested alignment %0 is not a positive power of two">;
def note_constexpr_alignment_too_big : Note<
  "requested alignment must be %0 or less for type %1; %2 is invalid">;
def note_constexpr_alignment_compute : Note<
  "cannot constant evaluate whether run-time alignment is at least %0">;
def note_constexpr_alignment_adjust : Note<
  "cannot constant evaluate the result of adjusting alignment to %0">;
def note_constexpr_destroy_out_of_lifetime : Note<
  "destroying object '%0' whose lifetime has already ended">;
def note_constexpr_unsupported_destruction : Note<
  "non-trivial destruction of type %0 in a constant expression is not supported">;
def note_constexpr_unsupported_temporary_nontrivial_dtor : Note<
  "non-trivial destruction of lifetime-extended temporary with type %0 "
  "used in the result of a constant expression is not yet supported">;
def note_constexpr_unsupported_unsized_array : Note<
  "array-to-pointer decay of array member without known bound is not supported">;
def note_constexpr_unsized_array_indexed : Note<
  "indexing of array without known bound is not allowed "
  "in a constant expression">;
def note_constexpr_memcmp_unsupported : Note<
  "constant evaluation of %0 between arrays of types %1 and %2 "
````
- **L265 EN**: Continues logic associated with callable symbol `pointer`.
  **L265 CN**: 继续与可调用符号 `pointer` 相关的逻辑。
- **L266 EN**: Adds a standalone statement or declaration: `"%plural{1:byte|:bytes}1">;`.
  **L266 CN**: 添加一条独立语句或声明：`"%plural{1:byte|:bytes}1">;`。
- **L267 EN**: Declares TableGen def record `note_constexpr_invalid_alignment`.
  **L267 CN**: 声明 TableGen def 记录 `note_constexpr_invalid_alignment`。
- **L268 EN**: Adds a standalone statement or declaration: `"requested alignment %0 is not a positive power of two">;`.
  **L268 CN**: 添加一条独立语句或声明：`"requested alignment %0 is not a positive power of two">;`。
- **L269 EN**: Declares TableGen def record `note_constexpr_alignment_too_big`.
  **L269 CN**: 声明 TableGen def 记录 `note_constexpr_alignment_too_big`。
- **L270 EN**: Adds a standalone statement or declaration: `"requested alignment must be %0 or less for type %1; %2 is invalid">;`.
  **L270 CN**: 添加一条独立语句或声明：`"requested alignment must be %0 or less for type %1; %2 is invalid">;`。
- **L271 EN**: Declares TableGen def record `note_constexpr_alignment_compute`.
  **L271 CN**: 声明 TableGen def 记录 `note_constexpr_alignment_compute`。
- **L272 EN**: Adds a standalone statement or declaration: `"cannot constant evaluate whether run-time alignment is at least %0">;`.
  **L272 CN**: 添加一条独立语句或声明：`"cannot constant evaluate whether run-time alignment is at least %0">;`。
- **L273 EN**: Declares TableGen def record `note_constexpr_alignment_adjust`.
  **L273 CN**: 声明 TableGen def 记录 `note_constexpr_alignment_adjust`。
- **L274 EN**: Adds a standalone statement or declaration: `"cannot constant evaluate the result of adjusting alignment to %0">;`.
  **L274 CN**: 添加一条独立语句或声明：`"cannot constant evaluate the result of adjusting alignment to %0">;`。
- **L275 EN**: Declares TableGen def record `note_constexpr_destroy_out_of_lifetime`.
  **L275 CN**: 声明 TableGen def 记录 `note_constexpr_destroy_out_of_lifetime`。
- **L276 EN**: Adds a standalone statement or declaration: `"destroying object '%0' whose lifetime has already ended">;`.
  **L276 CN**: 添加一条独立语句或声明：`"destroying object '%0' whose lifetime has already ended">;`。
- **L277 EN**: Declares TableGen def record `note_constexpr_unsupported_destruction`.
  **L277 CN**: 声明 TableGen def 记录 `note_constexpr_unsupported_destruction`。
- **L278 EN**: Adds a standalone statement or declaration: `"non-trivial destruction of type %0 in a constant expression is not supported">;`.
  **L278 CN**: 添加一条独立语句或声明：`"non-trivial destruction of type %0 in a constant expression is not supported">;`。
- **L279 EN**: Declares TableGen def record `note_constexpr_unsupported_temporary_nontrivial_dtor`.
  **L279 CN**: 声明 TableGen def 记录 `note_constexpr_unsupported_temporary_nontrivial_dtor`。
- **L280 EN**: Continues the surrounding expression or declaration: `"non-trivial destruction of lifetime-extended temporary with type %0 "`.
  **L280 CN**: 继续构造周围的表达式或声明：`"non-trivial destruction of lifetime-extended temporary with type %0 "`。
- **L281 EN**: Adds a standalone statement or declaration: `"used in the result of a constant expression is not yet supported">;`.
  **L281 CN**: 添加一条独立语句或声明：`"used in the result of a constant expression is not yet supported">;`。
- **L282 EN**: Declares TableGen def record `note_constexpr_unsupported_unsized_array`.
  **L282 CN**: 声明 TableGen def 记录 `note_constexpr_unsupported_unsized_array`。
- **L283 EN**: Adds a standalone statement or declaration: `"array-to-pointer decay of array member without known bound is not supported">;`.
  **L283 CN**: 添加一条独立语句或声明：`"array-to-pointer decay of array member without known bound is not supported">;`。
- **L284 EN**: Declares TableGen def record `note_constexpr_unsized_array_indexed`.
  **L284 CN**: 声明 TableGen def 记录 `note_constexpr_unsized_array_indexed`。
- **L285 EN**: Continues the surrounding expression or declaration: `"indexing of array without known bound is not allowed "`.
  **L285 CN**: 继续构造周围的表达式或声明：`"indexing of array without known bound is not allowed "`。
- **L286 EN**: Adds a standalone statement or declaration: `"in a constant expression">;`.
  **L286 CN**: 添加一条独立语句或声明：`"in a constant expression">;`。
- **L287 EN**: Declares TableGen def record `note_constexpr_memcmp_unsupported`.
  **L287 CN**: 声明 TableGen def 记录 `note_constexpr_memcmp_unsupported`。
- **L288 EN**: Continues the surrounding expression or declaration: `"constant evaluation of %0 between arrays of types %1 and %2 "`.
  **L288 CN**: 继续构造周围的表达式或声明：`"constant evaluation of %0 between arrays of types %1 and %2 "`。

### Lines 289-312

````tablegen
  "is not supported; only arrays of narrow character types can be compared">;
def note_constexpr_memchr_unsupported : Note<
  "constant evaluation of %0 on array of type %1 "
  "is not supported; only arrays of narrow character types can be searched">;
def note_constexpr_memcpy_null : Note<
  "%select{source|destination}2 of "
  "'%select{%select{memcpy|wmemcpy}1|%select{memmove|wmemmove}1}0' "
  "is %3">;
def note_constexpr_memcpy_type_pun : Note<
  "cannot constant evaluate '%select{memcpy|memmove}0' from object of "
  "type %1 to object of type %2">;
def note_constexpr_memcpy_nontrivial : Note<
  "cannot constant evaluate '%select{memcpy|memmove}0' between objects of "
  "non-trivially-copyable type %1">;
def note_constexpr_memcpy_incomplete_type : Note<
  "cannot constant evaluate '%select{memcpy|memmove}0' between objects of "
  "incomplete type %1">;
def note_constexpr_memcpy_overlap : Note<
  "'%select{memcpy|wmemcpy}0' between overlapping memory regions">;
def note_constexpr_memcpy_unsupported : Note<
  "'%select{%select{memcpy|wmemcpy}1|%select{memmove|wmemmove}1}0' "
  "not supported: %select{"
  "size to copy (%4) is not a multiple of size of element type %3 (%5)|"
  "source is not a contiguous array of at least %4 elements of type %3|"
````
- **L289 EN**: Adds a standalone statement or declaration: `"is not supported; only arrays of narrow character types can be compared">;`.
  **L289 CN**: 添加一条独立语句或声明：`"is not supported; only arrays of narrow character types can be compared">;`。
- **L290 EN**: Declares TableGen def record `note_constexpr_memchr_unsupported`.
  **L290 CN**: 声明 TableGen def 记录 `note_constexpr_memchr_unsupported`。
- **L291 EN**: Continues the surrounding expression or declaration: `"constant evaluation of %0 on array of type %1 "`.
  **L291 CN**: 继续构造周围的表达式或声明：`"constant evaluation of %0 on array of type %1 "`。
- **L292 EN**: Adds a standalone statement or declaration: `"is not supported; only arrays of narrow character types can be searched">;`.
  **L292 CN**: 添加一条独立语句或声明：`"is not supported; only arrays of narrow character types can be searched">;`。
- **L293 EN**: Declares TableGen def record `note_constexpr_memcpy_null`.
  **L293 CN**: 声明 TableGen def 记录 `note_constexpr_memcpy_null`。
- **L294 EN**: Continues the surrounding expression or declaration: `"%select{source|destination}2 of "`.
  **L294 CN**: 继续构造周围的表达式或声明：`"%select{source|destination}2 of "`。
- **L295 EN**: Continues the surrounding expression or declaration: `"'%select{%select{memcpy|wmemcpy}1|%select{memmove|wmemmove}1}0' "`.
  **L295 CN**: 继续构造周围的表达式或声明：`"'%select{%select{memcpy|wmemcpy}1|%select{memmove|wmemmove}1}0' "`。
- **L296 EN**: Adds a standalone statement or declaration: `"is %3">;`.
  **L296 CN**: 添加一条独立语句或声明：`"is %3">;`。
- **L297 EN**: Declares TableGen def record `note_constexpr_memcpy_type_pun`.
  **L297 CN**: 声明 TableGen def 记录 `note_constexpr_memcpy_type_pun`。
- **L298 EN**: Continues the surrounding expression or declaration: `"cannot constant evaluate '%select{memcpy|memmove}0' from object of "`.
  **L298 CN**: 继续构造周围的表达式或声明：`"cannot constant evaluate '%select{memcpy|memmove}0' from object of "`。
- **L299 EN**: Adds a standalone statement or declaration: `"type %1 to object of type %2">;`.
  **L299 CN**: 添加一条独立语句或声明：`"type %1 to object of type %2">;`。
- **L300 EN**: Declares TableGen def record `note_constexpr_memcpy_nontrivial`.
  **L300 CN**: 声明 TableGen def 记录 `note_constexpr_memcpy_nontrivial`。
- **L301 EN**: Continues the surrounding expression or declaration: `"cannot constant evaluate '%select{memcpy|memmove}0' between objects of "`.
  **L301 CN**: 继续构造周围的表达式或声明：`"cannot constant evaluate '%select{memcpy|memmove}0' between objects of "`。
- **L302 EN**: Adds a standalone statement or declaration: `"non-trivially-copyable type %1">;`.
  **L302 CN**: 添加一条独立语句或声明：`"non-trivially-copyable type %1">;`。
- **L303 EN**: Declares TableGen def record `note_constexpr_memcpy_incomplete_type`.
  **L303 CN**: 声明 TableGen def 记录 `note_constexpr_memcpy_incomplete_type`。
- **L304 EN**: Continues the surrounding expression or declaration: `"cannot constant evaluate '%select{memcpy|memmove}0' between objects of "`.
  **L304 CN**: 继续构造周围的表达式或声明：`"cannot constant evaluate '%select{memcpy|memmove}0' between objects of "`。
- **L305 EN**: Adds a standalone statement or declaration: `"incomplete type %1">;`.
  **L305 CN**: 添加一条独立语句或声明：`"incomplete type %1">;`。
- **L306 EN**: Declares TableGen def record `note_constexpr_memcpy_overlap`.
  **L306 CN**: 声明 TableGen def 记录 `note_constexpr_memcpy_overlap`。
- **L307 EN**: Adds a standalone statement or declaration: `"'%select{memcpy|wmemcpy}0' between overlapping memory regions">;`.
  **L307 CN**: 添加一条独立语句或声明：`"'%select{memcpy|wmemcpy}0' between overlapping memory regions">;`。
- **L308 EN**: Declares TableGen def record `note_constexpr_memcpy_unsupported`.
  **L308 CN**: 声明 TableGen def 记录 `note_constexpr_memcpy_unsupported`。
- **L309 EN**: Continues the surrounding expression or declaration: `"'%select{%select{memcpy|wmemcpy}1|%select{memmove|wmemmove}1}0' "`.
  **L309 CN**: 继续构造周围的表达式或声明：`"'%select{%select{memcpy|wmemcpy}1|%select{memmove|wmemmove}1}0' "`。
- **L310 EN**: Continues the surrounding expression or declaration: `"not supported: %select{"`.
  **L310 CN**: 继续构造周围的表达式或声明：`"not supported: %select{"`。
- **L311 EN**: Continues logic associated with callable symbol `copy`.
  **L311 CN**: 继续与可调用符号 `copy` 相关的逻辑。
- **L312 EN**: Continues the surrounding expression or declaration: `"source is not a contiguous array of at least %4 elements of type %3|"`.
  **L312 CN**: 继续构造周围的表达式或声明：`"source is not a contiguous array of at least %4 elements of type %3|"`。

### Lines 313-336

````tablegen
  "destination is not a contiguous array of at least %4 elements of type %3}2">;
def note_constexpr_bit_cast_unsupported_type : Note<
  "constexpr bit cast involving type %0 is not yet supported">;
def note_constexpr_bit_cast_unsupported_bitfield : Note<
  "constexpr bit_cast involving bit-field is not yet supported">;
def note_constexpr_bit_cast_invalid_type : Note<
  "bit_cast %select{from|to}0 a %select{|type with a }1"
  "%select{union|pointer|member pointer|volatile|reference}2 "
  "%select{type|member}1 is not allowed in a constant expression">;
def note_constexpr_bit_cast_invalid_subtype : Note<
  "invalid type %0 is a %select{member|base}1 of %2">;
def note_constexpr_bit_cast_invalid_vector : Note<
  "bit_cast involving type %0 is not allowed in a constant expression; "
  "element size %1 * element count %2 is not a multiple of the byte size %3">;
def note_constexpr_bit_cast_indet_dest : Note<
  "indeterminate value can only initialize an object of type 'unsigned char'"
  "%select{, 'char',|}1 or 'std::byte'; %0 is invalid">;
def note_constexpr_bit_cast_unrepresentable_value : Note<
  "value %1 cannot be represented in type %0">;
def note_constexpr_pseudo_destructor : Note<
  "pseudo-destructor call is not permitted in constant expressions "
  "until C++20">;
def note_constexpr_construct_complex_elem : Note<
  "construction of individual component of complex number is not yet supported "
````
- **L313 EN**: Adds a standalone statement or declaration: `"destination is not a contiguous array of at least %4 elements of type %3}2">;`.
  **L313 CN**: 添加一条独立语句或声明：`"destination is not a contiguous array of at least %4 elements of type %3}2">;`。
- **L314 EN**: Declares TableGen def record `note_constexpr_bit_cast_unsupported_type`.
  **L314 CN**: 声明 TableGen def 记录 `note_constexpr_bit_cast_unsupported_type`。
- **L315 EN**: Adds a standalone statement or declaration: `"constexpr bit cast involving type %0 is not yet supported">;`.
  **L315 CN**: 添加一条独立语句或声明：`"constexpr bit cast involving type %0 is not yet supported">;`。
- **L316 EN**: Declares TableGen def record `note_constexpr_bit_cast_unsupported_bitfield`.
  **L316 CN**: 声明 TableGen def 记录 `note_constexpr_bit_cast_unsupported_bitfield`。
- **L317 EN**: Adds a standalone statement or declaration: `"constexpr bit_cast involving bit-field is not yet supported">;`.
  **L317 CN**: 添加一条独立语句或声明：`"constexpr bit_cast involving bit-field is not yet supported">;`。
- **L318 EN**: Declares TableGen def record `note_constexpr_bit_cast_invalid_type`.
  **L318 CN**: 声明 TableGen def 记录 `note_constexpr_bit_cast_invalid_type`。
- **L319 EN**: Continues the surrounding expression or declaration: `"bit_cast %select{from|to}0 a %select{|type with a }1"`.
  **L319 CN**: 继续构造周围的表达式或声明：`"bit_cast %select{from|to}0 a %select{|type with a }1"`。
- **L320 EN**: Continues the surrounding expression or declaration: `"%select{union|pointer|member pointer|volatile|reference}2 "`.
  **L320 CN**: 继续构造周围的表达式或声明：`"%select{union|pointer|member pointer|volatile|reference}2 "`。
- **L321 EN**: Adds a standalone statement or declaration: `"%select{type|member}1 is not allowed in a constant expression">;`.
  **L321 CN**: 添加一条独立语句或声明：`"%select{type|member}1 is not allowed in a constant expression">;`。
- **L322 EN**: Declares TableGen def record `note_constexpr_bit_cast_invalid_subtype`.
  **L322 CN**: 声明 TableGen def 记录 `note_constexpr_bit_cast_invalid_subtype`。
- **L323 EN**: Adds a standalone statement or declaration: `"invalid type %0 is a %select{member|base}1 of %2">;`.
  **L323 CN**: 添加一条独立语句或声明：`"invalid type %0 is a %select{member|base}1 of %2">;`。
- **L324 EN**: Declares TableGen def record `note_constexpr_bit_cast_invalid_vector`.
  **L324 CN**: 声明 TableGen def 记录 `note_constexpr_bit_cast_invalid_vector`。
- **L325 EN**: Continues the surrounding expression or declaration: `"bit_cast involving type %0 is not allowed in a constant expression; "`.
  **L325 CN**: 继续构造周围的表达式或声明：`"bit_cast involving type %0 is not allowed in a constant expression; "`。
- **L326 EN**: Adds a standalone statement or declaration: `"element size %1 * element count %2 is not a multiple of the byte size %3">;`.
  **L326 CN**: 添加一条独立语句或声明：`"element size %1 * element count %2 is not a multiple of the byte size %3">;`。
- **L327 EN**: Declares TableGen def record `note_constexpr_bit_cast_indet_dest`.
  **L327 CN**: 声明 TableGen def 记录 `note_constexpr_bit_cast_indet_dest`。
- **L328 EN**: Continues the surrounding expression or declaration: `"indeterminate value can only initialize an object of type 'unsigned char'"`.
  **L328 CN**: 继续构造周围的表达式或声明：`"indeterminate value can only initialize an object of type 'unsigned char'"`。
- **L329 EN**: Adds a standalone statement or declaration: `"%select{, 'char',|}1 or 'std::byte'; %0 is invalid">;`.
  **L329 CN**: 添加一条独立语句或声明：`"%select{, 'char',|}1 or 'std::byte'; %0 is invalid">;`。
- **L330 EN**: Declares TableGen def record `note_constexpr_bit_cast_unrepresentable_value`.
  **L330 CN**: 声明 TableGen def 记录 `note_constexpr_bit_cast_unrepresentable_value`。
- **L331 EN**: Adds a standalone statement or declaration: `"value %1 cannot be represented in type %0">;`.
  **L331 CN**: 添加一条独立语句或声明：`"value %1 cannot be represented in type %0">;`。
- **L332 EN**: Declares TableGen def record `note_constexpr_pseudo_destructor`.
  **L332 CN**: 声明 TableGen def 记录 `note_constexpr_pseudo_destructor`。
- **L333 EN**: Continues the surrounding expression or declaration: `"pseudo-destructor call is not permitted in constant expressions "`.
  **L333 CN**: 继续构造周围的表达式或声明：`"pseudo-destructor call is not permitted in constant expressions "`。
- **L334 EN**: Adds a standalone statement or declaration: `"until C++20">;`.
  **L334 CN**: 添加一条独立语句或声明：`"until C++20">;`。
- **L335 EN**: Declares TableGen def record `note_constexpr_construct_complex_elem`.
  **L335 CN**: 声明 TableGen def 记录 `note_constexpr_construct_complex_elem`。
- **L336 EN**: Continues the surrounding expression or declaration: `"construction of individual component of complex number is not yet supported "`.
  **L336 CN**: 继续构造周围的表达式或声明：`"construction of individual component of complex number is not yet supported "`。

### Lines 337-360

````tablegen
  "in constant expressions">;
def note_constexpr_destroy_complex_elem : Note<
  "destruction of individual component of complex number is not yet supported "
  "in constant expressions">;
def note_constexpr_new : Note<
  "dynamic memory allocation is not permitted in constant expressions "
  "until C++20">;
def note_constexpr_new_non_replaceable : Note<
  "call to %select{placement|class-specific}0 %1">;
def note_constexpr_new_placement : Note<
  "this placement new expression is not supported in constant expressions "
  "%select{|before C++2c}0">;
def note_constexpr_placement_new_wrong_type : Note<
  "placement new would change type of storage from %0 to %1">;
def note_constexpr_new_negative : Note<
  "cannot allocate array; evaluated array bound %0 is negative">;
def note_constexpr_new_too_large : Note<
  "cannot allocate array; evaluated array bound %0 is too large">;
def note_constexpr_new_exceeds_limits : Note<
  "cannot allocate array; evaluated array bound %0 exceeds the limit (%1); "
  "use '-fconstexpr-steps' to increase this limit">;
def note_constexpr_new_too_small : Note<
  "cannot allocate array; evaluated array bound %0 is too small to hold "
  "%1 explicitly initialized elements">;
````
- **L337 EN**: Adds a standalone statement or declaration: `"in constant expressions">;`.
  **L337 CN**: 添加一条独立语句或声明：`"in constant expressions">;`。
- **L338 EN**: Declares TableGen def record `note_constexpr_destroy_complex_elem`.
  **L338 CN**: 声明 TableGen def 记录 `note_constexpr_destroy_complex_elem`。
- **L339 EN**: Continues the surrounding expression or declaration: `"destruction of individual component of complex number is not yet supported "`.
  **L339 CN**: 继续构造周围的表达式或声明：`"destruction of individual component of complex number is not yet supported "`。
- **L340 EN**: Adds a standalone statement or declaration: `"in constant expressions">;`.
  **L340 CN**: 添加一条独立语句或声明：`"in constant expressions">;`。
- **L341 EN**: Declares TableGen def record `note_constexpr_new`.
  **L341 CN**: 声明 TableGen def 记录 `note_constexpr_new`。
- **L342 EN**: Continues the surrounding expression or declaration: `"dynamic memory allocation is not permitted in constant expressions "`.
  **L342 CN**: 继续构造周围的表达式或声明：`"dynamic memory allocation is not permitted in constant expressions "`。
- **L343 EN**: Adds a standalone statement or declaration: `"until C++20">;`.
  **L343 CN**: 添加一条独立语句或声明：`"until C++20">;`。
- **L344 EN**: Declares TableGen def record `note_constexpr_new_non_replaceable`.
  **L344 CN**: 声明 TableGen def 记录 `note_constexpr_new_non_replaceable`。
- **L345 EN**: Adds a standalone statement or declaration: `"call to %select{placement|class-specific}0 %1">;`.
  **L345 CN**: 添加一条独立语句或声明：`"call to %select{placement|class-specific}0 %1">;`。
- **L346 EN**: Declares TableGen def record `note_constexpr_new_placement`.
  **L346 CN**: 声明 TableGen def 记录 `note_constexpr_new_placement`。
- **L347 EN**: Continues the surrounding expression or declaration: `"this placement new expression is not supported in constant expressions "`.
  **L347 CN**: 继续构造周围的表达式或声明：`"this placement new expression is not supported in constant expressions "`。
- **L348 EN**: Adds a standalone statement or declaration: `"%select{|before C++2c}0">;`.
  **L348 CN**: 添加一条独立语句或声明：`"%select{|before C++2c}0">;`。
- **L349 EN**: Declares TableGen def record `note_constexpr_placement_new_wrong_type`.
  **L349 CN**: 声明 TableGen def 记录 `note_constexpr_placement_new_wrong_type`。
- **L350 EN**: Adds a standalone statement or declaration: `"placement new would change type of storage from %0 to %1">;`.
  **L350 CN**: 添加一条独立语句或声明：`"placement new would change type of storage from %0 to %1">;`。
- **L351 EN**: Declares TableGen def record `note_constexpr_new_negative`.
  **L351 CN**: 声明 TableGen def 记录 `note_constexpr_new_negative`。
- **L352 EN**: Adds a standalone statement or declaration: `"cannot allocate array; evaluated array bound %0 is negative">;`.
  **L352 CN**: 添加一条独立语句或声明：`"cannot allocate array; evaluated array bound %0 is negative">;`。
- **L353 EN**: Declares TableGen def record `note_constexpr_new_too_large`.
  **L353 CN**: 声明 TableGen def 记录 `note_constexpr_new_too_large`。
- **L354 EN**: Adds a standalone statement or declaration: `"cannot allocate array; evaluated array bound %0 is too large">;`.
  **L354 CN**: 添加一条独立语句或声明：`"cannot allocate array; evaluated array bound %0 is too large">;`。
- **L355 EN**: Declares TableGen def record `note_constexpr_new_exceeds_limits`.
  **L355 CN**: 声明 TableGen def 记录 `note_constexpr_new_exceeds_limits`。
- **L356 EN**: Continues logic associated with callable symbol `limit`.
  **L356 CN**: 继续与可调用符号 `limit` 相关的逻辑。
- **L357 EN**: Adds a standalone statement or declaration: `"use '-fconstexpr-steps' to increase this limit">;`.
  **L357 CN**: 添加一条独立语句或声明：`"use '-fconstexpr-steps' to increase this limit">;`。
- **L358 EN**: Declares TableGen def record `note_constexpr_new_too_small`.
  **L358 CN**: 声明 TableGen def 记录 `note_constexpr_new_too_small`。
- **L359 EN**: Continues the surrounding expression or declaration: `"cannot allocate array; evaluated array bound %0 is too small to hold "`.
  **L359 CN**: 继续构造周围的表达式或声明：`"cannot allocate array; evaluated array bound %0 is too small to hold "`。
- **L360 EN**: Adds a standalone statement or declaration: `"%1 explicitly initialized elements">;`.
  **L360 CN**: 添加一条独立语句或声明：`"%1 explicitly initialized elements">;`。

### Lines 361-384

````tablegen
def note_constexpr_new_untyped : Note<
  "cannot allocate untyped memory in a constant expression; "
  "use 'std::allocator<T>::allocate' to allocate memory of type 'T'">;
def note_constexpr_new_not_complete_object_type : Note<
  "cannot allocate memory of %select{incomplete|function}0 type %1">;
def note_constexpr_operator_new_bad_size : Note<
  "allocated size %0 is not a multiple of size %1 of element type %2">;
def note_constexpr_delete_not_heap_alloc : Note<
  "delete of pointer '%0' that does not point to a heap-allocated object">;
def note_constexpr_double_delete : Note<
  "delete of pointer that has already been deleted">;
def note_constexpr_double_destroy : Note<
  "destruction of object that is already being destroyed">;
def note_constexpr_new_delete_mismatch : Note<
  "%plural{2:'delete' used to delete pointer to object "
  "allocated with 'std::allocator<...>::allocate'|"
  ":%select{non-array delete|array delete|'std::allocator<...>::deallocate'}0 "
  "used to delete pointer to "
  "%select{array object of type %2|non-array object of type %2|"
  "object allocated with 'new'}0}1">;
def note_constexpr_deallocate_null : Note<
  "'std::allocator<...>::deallocate' used to delete a null pointer">;
def note_constexpr_delete_subobject : Note<
  "delete of pointer%select{ to subobject|}1 '%0' "
````
- **L361 EN**: Declares TableGen def record `note_constexpr_new_untyped`.
  **L361 CN**: 声明 TableGen def 记录 `note_constexpr_new_untyped`。
- **L362 EN**: Continues the surrounding expression or declaration: `"cannot allocate untyped memory in a constant expression; "`.
  **L362 CN**: 继续构造周围的表达式或声明：`"cannot allocate untyped memory in a constant expression; "`。
- **L363 EN**: Adds a standalone statement or declaration: `"use 'std::allocator<T>::allocate' to allocate memory of type 'T'">;`.
  **L363 CN**: 添加一条独立语句或声明：`"use 'std::allocator<T>::allocate' to allocate memory of type 'T'">;`。
- **L364 EN**: Declares TableGen def record `note_constexpr_new_not_complete_object_type`.
  **L364 CN**: 声明 TableGen def 记录 `note_constexpr_new_not_complete_object_type`。
- **L365 EN**: Adds a standalone statement or declaration: `"cannot allocate memory of %select{incomplete|function}0 type %1">;`.
  **L365 CN**: 添加一条独立语句或声明：`"cannot allocate memory of %select{incomplete|function}0 type %1">;`。
- **L366 EN**: Declares TableGen def record `note_constexpr_operator_new_bad_size`.
  **L366 CN**: 声明 TableGen def 记录 `note_constexpr_operator_new_bad_size`。
- **L367 EN**: Adds a standalone statement or declaration: `"allocated size %0 is not a multiple of size %1 of element type %2">;`.
  **L367 CN**: 添加一条独立语句或声明：`"allocated size %0 is not a multiple of size %1 of element type %2">;`。
- **L368 EN**: Declares TableGen def record `note_constexpr_delete_not_heap_alloc`.
  **L368 CN**: 声明 TableGen def 记录 `note_constexpr_delete_not_heap_alloc`。
- **L369 EN**: Adds a standalone statement or declaration: `"delete of pointer '%0' that does not point to a heap-allocated object">;`.
  **L369 CN**: 添加一条独立语句或声明：`"delete of pointer '%0' that does not point to a heap-allocated object">;`。
- **L370 EN**: Declares TableGen def record `note_constexpr_double_delete`.
  **L370 CN**: 声明 TableGen def 记录 `note_constexpr_double_delete`。
- **L371 EN**: Adds a standalone statement or declaration: `"delete of pointer that has already been deleted">;`.
  **L371 CN**: 添加一条独立语句或声明：`"delete of pointer that has already been deleted">;`。
- **L372 EN**: Declares TableGen def record `note_constexpr_double_destroy`.
  **L372 CN**: 声明 TableGen def 记录 `note_constexpr_double_destroy`。
- **L373 EN**: Adds a standalone statement or declaration: `"destruction of object that is already being destroyed">;`.
  **L373 CN**: 添加一条独立语句或声明：`"destruction of object that is already being destroyed">;`。
- **L374 EN**: Declares TableGen def record `note_constexpr_new_delete_mismatch`.
  **L374 CN**: 声明 TableGen def 记录 `note_constexpr_new_delete_mismatch`。
- **L375 EN**: Continues the surrounding expression or declaration: `"%plural{2:'delete' used to delete pointer to object "`.
  **L375 CN**: 继续构造周围的表达式或声明：`"%plural{2:'delete' used to delete pointer to object "`。
- **L376 EN**: Continues the surrounding expression or declaration: `"allocated with 'std::allocator<...>::allocate'|"`.
  **L376 CN**: 继续构造周围的表达式或声明：`"allocated with 'std::allocator<...>::allocate'|"`。
- **L377 EN**: Continues the surrounding expression or declaration: `":%select{non-array delete|array delete|'std::allocator<...>::deallocate'}0 "`.
  **L377 CN**: 继续构造周围的表达式或声明：`":%select{non-array delete|array delete|'std::allocator<...>::deallocate'}0 "`。
- **L378 EN**: Continues the surrounding expression or declaration: `"used to delete pointer to "`.
  **L378 CN**: 继续构造周围的表达式或声明：`"used to delete pointer to "`。
- **L379 EN**: Continues the surrounding expression or declaration: `"%select{array object of type %2|non-array object of type %2|"`.
  **L379 CN**: 继续构造周围的表达式或声明：`"%select{array object of type %2|non-array object of type %2|"`。
- **L380 EN**: Adds a standalone statement or declaration: `"object allocated with 'new'}0}1">;`.
  **L380 CN**: 添加一条独立语句或声明：`"object allocated with 'new'}0}1">;`。
- **L381 EN**: Declares TableGen def record `note_constexpr_deallocate_null`.
  **L381 CN**: 声明 TableGen def 记录 `note_constexpr_deallocate_null`。
- **L382 EN**: Adds a standalone statement or declaration: `"'std::allocator<...>::deallocate' used to delete a null pointer">;`.
  **L382 CN**: 添加一条独立语句或声明：`"'std::allocator<...>::deallocate' used to delete a null pointer">;`。
- **L383 EN**: Declares TableGen def record `note_constexpr_delete_subobject`.
  **L383 CN**: 声明 TableGen def 记录 `note_constexpr_delete_subobject`。
- **L384 EN**: Continues the surrounding expression or declaration: `"delete of pointer%select{ to subobject|}1 '%0' "`.
  **L384 CN**: 继续构造周围的表达式或声明：`"delete of pointer%select{ to subobject|}1 '%0' "`。

### Lines 385-408

````tablegen
  "%select{|that does not point to complete object}1">;
def note_constexpr_delete_base_nonvirt_dtor : Note<
  "delete of object with dynamic type %1 through pointer to "
  "base class type %0 with non-virtual destructor">;
def note_constexpr_memory_leak : Note<
  "allocation performed here was not deallocated"
  "%plural{0:|: (along with %0 other memory leak%s0)}0">;
def note_constexpr_unsupported_layout : Note<
  "type %0 has unexpected layout">;
def note_constexpr_unsupported_flexible_array : Note<
  "flexible array initialization is not yet supported">;
def note_constexpr_non_const_vectorelements : Note<
  "cannot determine number of elements for sizeless vectors in a constant expression">;
def note_constexpr_assumption_failed : Note<
  "assumption evaluated to false">;
def note_constexpr_countzeroes_zero : Note<
  "evaluation of %select{__builtin_elementwise_clzg|__builtin_elementwise_ctzg}0 "
  "with a zero value is undefined">;
def note_constexpr_infer_alloc_token_type_inference_failed : Note<
  "could not infer allocation type for __builtin_infer_alloc_token">;
def note_constexpr_infer_alloc_token_no_metadata : Note<
  "could not get token metadata for inferred type">;
def note_constexpr_infer_alloc_token_stateful_mode
    : Note<"stateful alloc token mode not supported in constexpr">;
````
- **L385 EN**: Adds a standalone statement or declaration: `"%select{|that does not point to complete object}1">;`.
  **L385 CN**: 添加一条独立语句或声明：`"%select{|that does not point to complete object}1">;`。
- **L386 EN**: Declares TableGen def record `note_constexpr_delete_base_nonvirt_dtor`.
  **L386 CN**: 声明 TableGen def 记录 `note_constexpr_delete_base_nonvirt_dtor`。
- **L387 EN**: Continues the surrounding expression or declaration: `"delete of object with dynamic type %1 through pointer to "`.
  **L387 CN**: 继续构造周围的表达式或声明：`"delete of object with dynamic type %1 through pointer to "`。
- **L388 EN**: Adds a standalone statement or declaration: `"base class type %0 with non-virtual destructor">;`.
  **L388 CN**: 添加一条独立语句或声明：`"base class type %0 with non-virtual destructor">;`。
- **L389 EN**: Declares TableGen def record `note_constexpr_memory_leak`.
  **L389 CN**: 声明 TableGen def 记录 `note_constexpr_memory_leak`。
- **L390 EN**: Continues the surrounding expression or declaration: `"allocation performed here was not deallocated"`.
  **L390 CN**: 继续构造周围的表达式或声明：`"allocation performed here was not deallocated"`。
- **L391 EN**: Executes a call or declaration centered on `"%plural{0:|:`.
  **L391 CN**: 执行以 `"%plural{0:|:` 为核心的调用或声明。
- **L392 EN**: Declares TableGen def record `note_constexpr_unsupported_layout`.
  **L392 CN**: 声明 TableGen def 记录 `note_constexpr_unsupported_layout`。
- **L393 EN**: Adds a standalone statement or declaration: `"type %0 has unexpected layout">;`.
  **L393 CN**: 添加一条独立语句或声明：`"type %0 has unexpected layout">;`。
- **L394 EN**: Declares TableGen def record `note_constexpr_unsupported_flexible_array`.
  **L394 CN**: 声明 TableGen def 记录 `note_constexpr_unsupported_flexible_array`。
- **L395 EN**: Adds a standalone statement or declaration: `"flexible array initialization is not yet supported">;`.
  **L395 CN**: 添加一条独立语句或声明：`"flexible array initialization is not yet supported">;`。
- **L396 EN**: Declares TableGen def record `note_constexpr_non_const_vectorelements`.
  **L396 CN**: 声明 TableGen def 记录 `note_constexpr_non_const_vectorelements`。
- **L397 EN**: Adds a standalone statement or declaration: `"cannot determine number of elements for sizeless vectors in a constant expression">;`.
  **L397 CN**: 添加一条独立语句或声明：`"cannot determine number of elements for sizeless vectors in a constant expression">;`。
- **L398 EN**: Declares TableGen def record `note_constexpr_assumption_failed`.
  **L398 CN**: 声明 TableGen def 记录 `note_constexpr_assumption_failed`。
- **L399 EN**: Adds a standalone statement or declaration: `"assumption evaluated to false">;`.
  **L399 CN**: 添加一条独立语句或声明：`"assumption evaluated to false">;`。
- **L400 EN**: Declares TableGen def record `note_constexpr_countzeroes_zero`.
  **L400 CN**: 声明 TableGen def 记录 `note_constexpr_countzeroes_zero`。
- **L401 EN**: Continues the surrounding expression or declaration: `"evaluation of %select{__builtin_elementwise_clzg|__builtin_elementwise_ctzg}0 "`.
  **L401 CN**: 继续构造周围的表达式或声明：`"evaluation of %select{__builtin_elementwise_clzg|__builtin_elementwise_ctzg}0 "`。
- **L402 EN**: Adds a standalone statement or declaration: `"with a zero value is undefined">;`.
  **L402 CN**: 添加一条独立语句或声明：`"with a zero value is undefined">;`。
- **L403 EN**: Declares TableGen def record `note_constexpr_infer_alloc_token_type_inference_failed`.
  **L403 CN**: 声明 TableGen def 记录 `note_constexpr_infer_alloc_token_type_inference_failed`。
- **L404 EN**: Adds a standalone statement or declaration: `"could not infer allocation type for __builtin_infer_alloc_token">;`.
  **L404 CN**: 添加一条独立语句或声明：`"could not infer allocation type for __builtin_infer_alloc_token">;`。
- **L405 EN**: Declares TableGen def record `note_constexpr_infer_alloc_token_no_metadata`.
  **L405 CN**: 声明 TableGen def 记录 `note_constexpr_infer_alloc_token_no_metadata`。
- **L406 EN**: Adds a standalone statement or declaration: `"could not get token metadata for inferred type">;`.
  **L406 CN**: 添加一条独立语句或声明：`"could not get token metadata for inferred type">;`。
- **L407 EN**: Declares TableGen def record `note_constexpr_infer_alloc_token_stateful_mode`.
  **L407 CN**: 声明 TableGen def 记录 `note_constexpr_infer_alloc_token_stateful_mode`。
- **L408 EN**: Adds a standalone statement or declaration: `: Note<"stateful alloc token mode not supported in constexpr">;`.
  **L408 CN**: 添加一条独立语句或声明：`: Note<"stateful alloc token mode not supported in constexpr">;`。

### Lines 409-432

````tablegen

def warn_attribute_needs_aggregate : Warning<
  "%0 attribute is ignored in non-aggregate type %1">,
  InGroup<IgnoredAttributes>;

def warn_cxx20_compat_requires_explicit_init_non_aggregate : Warning<
  "explicit initialization of field %1 will not be enforced in C++20 and later "
  "because %2 has a user-declared constructor, making the type no longer an "
  "aggregate">,
  DefaultIgnore, InGroup<CXX20Compat>;

def warn_integer_constant_overflow : Warning<
  "overflow in expression; result is %0 with type %1">,
  InGroup<DiagGroup<"integer-overflow">>;
def warn_fixedpoint_constant_overflow : Warning<
  "overflow in expression; result is %0 with type %1">,
  InGroup<DiagGroup<"fixed-point-overflow">>;
def note_constexpr_unscoped_enum_out_of_range : Note<
  "integer value %0 is outside the valid range of values [%1, %2] for the "
  "enumeration type %3">;

def warn_is_constant_evaluated_always_true_constexpr : Warning<
  "'%0' will always evaluate to 'true' in a manifestly constant-evaluated expression">,
  InGroup<DiagGroup<"constant-evaluated">>;
````
- **L409 EN**: Blank line separating nearby declarations or logic blocks.
  **L409 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L410 EN**: Declares TableGen def record `warn_attribute_needs_aggregate`.
  **L410 CN**: 声明 TableGen def 记录 `warn_attribute_needs_aggregate`。
- **L411 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"%0 attribute is ignored in non-aggregate type %1">,`.
  **L411 CN**: 继续一个多行参数列表、初始化器或聚合项：`"%0 attribute is ignored in non-aggregate type %1">,`。
- **L412 EN**: Adds a standalone statement or declaration: `InGroup<IgnoredAttributes>;`.
  **L412 CN**: 添加一条独立语句或声明：`InGroup<IgnoredAttributes>;`。
- **L413 EN**: Blank line separating nearby declarations or logic blocks.
  **L413 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L414 EN**: Declares TableGen def record `warn_cxx20_compat_requires_explicit_init_non_aggregate`.
  **L414 CN**: 声明 TableGen def 记录 `warn_cxx20_compat_requires_explicit_init_non_aggregate`。
- **L415 EN**: Continues the surrounding expression or declaration: `"explicit initialization of field %1 will not be enforced in C++20 and later "`.
  **L415 CN**: 继续构造周围的表达式或声明：`"explicit initialization of field %1 will not be enforced in C++20 and later "`。
- **L416 EN**: Continues the surrounding expression or declaration: `"because %2 has a user-declared constructor, making the type no longer an "`.
  **L416 CN**: 继续构造周围的表达式或声明：`"because %2 has a user-declared constructor, making the type no longer an "`。
- **L417 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"aggregate">,`.
  **L417 CN**: 继续一个多行参数列表、初始化器或聚合项：`"aggregate">,`。
- **L418 EN**: Adds a standalone statement or declaration: `DefaultIgnore, InGroup<CXX20Compat>;`.
  **L418 CN**: 添加一条独立语句或声明：`DefaultIgnore, InGroup<CXX20Compat>;`。
- **L419 EN**: Blank line separating nearby declarations or logic blocks.
  **L419 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L420 EN**: Declares TableGen def record `warn_integer_constant_overflow`.
  **L420 CN**: 声明 TableGen def 记录 `warn_integer_constant_overflow`。
- **L421 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"overflow in expression; result is %0 with type %1">,`.
  **L421 CN**: 继续一个多行参数列表、初始化器或聚合项：`"overflow in expression; result is %0 with type %1">,`。
- **L422 EN**: Adds a standalone statement or declaration: `InGroup<DiagGroup<"integer-overflow">>;`.
  **L422 CN**: 添加一条独立语句或声明：`InGroup<DiagGroup<"integer-overflow">>;`。
- **L423 EN**: Declares TableGen def record `warn_fixedpoint_constant_overflow`.
  **L423 CN**: 声明 TableGen def 记录 `warn_fixedpoint_constant_overflow`。
- **L424 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"overflow in expression; result is %0 with type %1">,`.
  **L424 CN**: 继续一个多行参数列表、初始化器或聚合项：`"overflow in expression; result is %0 with type %1">,`。
- **L425 EN**: Adds a standalone statement or declaration: `InGroup<DiagGroup<"fixed-point-overflow">>;`.
  **L425 CN**: 添加一条独立语句或声明：`InGroup<DiagGroup<"fixed-point-overflow">>;`。
- **L426 EN**: Declares TableGen def record `note_constexpr_unscoped_enum_out_of_range`.
  **L426 CN**: 声明 TableGen def 记录 `note_constexpr_unscoped_enum_out_of_range`。
- **L427 EN**: Continues the surrounding expression or declaration: `"integer value %0 is outside the valid range of values [%1, %2] for the "`.
  **L427 CN**: 继续构造周围的表达式或声明：`"integer value %0 is outside the valid range of values [%1, %2] for the "`。
- **L428 EN**: Adds a standalone statement or declaration: `"enumeration type %3">;`.
  **L428 CN**: 添加一条独立语句或声明：`"enumeration type %3">;`。
- **L429 EN**: Blank line separating nearby declarations or logic blocks.
  **L429 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L430 EN**: Declares TableGen def record `warn_is_constant_evaluated_always_true_constexpr`.
  **L430 CN**: 声明 TableGen def 记录 `warn_is_constant_evaluated_always_true_constexpr`。
- **L431 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'%0' will always evaluate to 'true' in a manifestly constant-evaluated expression">,`.
  **L431 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'%0' will always evaluate to 'true' in a manifestly constant-evaluated expression">,`。
- **L432 EN**: Adds a standalone statement or declaration: `InGroup<DiagGroup<"constant-evaluated">>;`.
  **L432 CN**: 添加一条独立语句或声明：`InGroup<DiagGroup<"constant-evaluated">>;`。

### Lines 433-456

````tablegen

def err_invalid_is_within_lifetime : Note<
  "'%0' cannot be called with "
  "%select{a null pointer|a one-past-the-end pointer|"
  "a pointer to an object whose lifetime has not yet begun}1"
>;

// inline asm related.
let CategoryName = "Inline Assembly Issue" in {
  def err_asm_invalid_escape : Error<
    "invalid %% escape in inline assembly string">;
  def err_asm_unknown_symbolic_operand_name : Error<
    "unknown symbolic operand name in inline assembly string">;

  def err_asm_unterminated_symbolic_operand_name : Error<
    "unterminated symbolic operand name in inline assembly string">;
  def err_asm_empty_symbolic_operand_name : Error<
    "empty symbolic operand name in inline assembly string">;
  def err_asm_invalid_operand_number : Error<
    "invalid operand number in inline asm string">;
}

// vtable related.
let CategoryName = "VTable ABI Issue" in {
````
- **L433 EN**: Blank line separating nearby declarations or logic blocks.
  **L433 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L434 EN**: Declares TableGen def record `err_invalid_is_within_lifetime`.
  **L434 CN**: 声明 TableGen def 记录 `err_invalid_is_within_lifetime`。
- **L435 EN**: Continues the surrounding expression or declaration: `"'%0' cannot be called with "`.
  **L435 CN**: 继续构造周围的表达式或声明：`"'%0' cannot be called with "`。
- **L436 EN**: Continues the surrounding expression or declaration: `"%select{a null pointer|a one-past-the-end pointer|"`.
  **L436 CN**: 继续构造周围的表达式或声明：`"%select{a null pointer|a one-past-the-end pointer|"`。
- **L437 EN**: Continues the surrounding expression or declaration: `"a pointer to an object whose lifetime has not yet begun}1"`.
  **L437 CN**: 继续构造周围的表达式或声明：`"a pointer to an object whose lifetime has not yet begun}1"`。
- **L438 EN**: Adds a standalone statement or declaration: `>;`.
  **L438 CN**: 添加一条独立语句或声明：`>;`。
- **L439 EN**: Blank line separating nearby declarations or logic blocks.
  **L439 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L440 EN**: Comment explains nearby logic, constraints, or intent: `inline asm related.`.
  **L440 CN**: 注释解释附近代码的逻辑、约束或设计意图：`inline asm related.`。
- **L441 EN**: Assigns a TableGen property that affects following records or inherited fields: `let CategoryName = "Inline Assembly Issue" in {`.
  **L441 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let CategoryName = "Inline Assembly Issue" in {`。
- **L442 EN**: Declares TableGen def record `err_asm_invalid_escape`.
  **L442 CN**: 声明 TableGen def 记录 `err_asm_invalid_escape`。
- **L443 EN**: Adds a standalone statement or declaration: `"invalid %% escape in inline assembly string">;`.
  **L443 CN**: 添加一条独立语句或声明：`"invalid %% escape in inline assembly string">;`。
- **L444 EN**: Declares TableGen def record `err_asm_unknown_symbolic_operand_name`.
  **L444 CN**: 声明 TableGen def 记录 `err_asm_unknown_symbolic_operand_name`。
- **L445 EN**: Adds a standalone statement or declaration: `"unknown symbolic operand name in inline assembly string">;`.
  **L445 CN**: 添加一条独立语句或声明：`"unknown symbolic operand name in inline assembly string">;`。
- **L446 EN**: Blank line separating nearby declarations or logic blocks.
  **L446 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L447 EN**: Declares TableGen def record `err_asm_unterminated_symbolic_operand_name`.
  **L447 CN**: 声明 TableGen def 记录 `err_asm_unterminated_symbolic_operand_name`。
- **L448 EN**: Adds a standalone statement or declaration: `"unterminated symbolic operand name in inline assembly string">;`.
  **L448 CN**: 添加一条独立语句或声明：`"unterminated symbolic operand name in inline assembly string">;`。
- **L449 EN**: Declares TableGen def record `err_asm_empty_symbolic_operand_name`.
  **L449 CN**: 声明 TableGen def 记录 `err_asm_empty_symbolic_operand_name`。
- **L450 EN**: Adds a standalone statement or declaration: `"empty symbolic operand name in inline assembly string">;`.
  **L450 CN**: 添加一条独立语句或声明：`"empty symbolic operand name in inline assembly string">;`。
- **L451 EN**: Declares TableGen def record `err_asm_invalid_operand_number`.
  **L451 CN**: 声明 TableGen def 记录 `err_asm_invalid_operand_number`。
- **L452 EN**: Adds a standalone statement or declaration: `"invalid operand number in inline asm string">;`.
  **L452 CN**: 添加一条独立语句或声明：`"invalid operand number in inline asm string">;`。
- **L453 EN**: Closes the current lexical scope or compound statement.
  **L453 CN**: 结束当前词法作用域或复合语句块。
- **L454 EN**: Blank line separating nearby declarations or logic blocks.
  **L454 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L455 EN**: Comment explains nearby logic, constraints, or intent: `vtable related.`.
  **L455 CN**: 注释解释附近代码的逻辑、约束或设计意图：`vtable related.`。
- **L456 EN**: Assigns a TableGen property that affects following records or inherited fields: `let CategoryName = "VTable ABI Issue" in {`.
  **L456 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let CategoryName = "VTable ABI Issue" in {`。

### Lines 457-480

````tablegen
  def err_vftable_ambiguous_component : Error<
    "ambiguous vftable component for %0 introduced via covariant thunks; "
    "this is an inherent limitation of the ABI">;
  def note_covariant_thunk : Note<
    "covariant thunk required by %0">;
}

// Importing ASTs
def err_odr_variable_type_inconsistent : Error<
  "external variable %0 declared with incompatible types in different "
  "translation units (%1 vs. %2)">;
def warn_odr_variable_type_inconsistent : Warning<
  "external variable %0 declared with incompatible types in different "
  "translation units (%1 vs. %2)">,
  InGroup<ODR>;
def err_odr_variable_multiple_def : Error<
  "external variable %0 defined in multiple translation units">;
def warn_odr_variable_multiple_def : Warning<
  "external variable %0 defined in multiple translation units">,
  InGroup<ODR>;
def note_odr_value_here : Note<"declared here with type %0">;
def err_odr_function_type_inconsistent : Error<
  "external function %0 declared with incompatible types in different "
  "translation units (%1 vs. %2)">;
````
- **L457 EN**: Declares TableGen def record `err_vftable_ambiguous_component`.
  **L457 CN**: 声明 TableGen def 记录 `err_vftable_ambiguous_component`。
- **L458 EN**: Continues the surrounding expression or declaration: `"ambiguous vftable component for %0 introduced via covariant thunks; "`.
  **L458 CN**: 继续构造周围的表达式或声明：`"ambiguous vftable component for %0 introduced via covariant thunks; "`。
- **L459 EN**: Adds a standalone statement or declaration: `"this is an inherent limitation of the ABI">;`.
  **L459 CN**: 添加一条独立语句或声明：`"this is an inherent limitation of the ABI">;`。
- **L460 EN**: Declares TableGen def record `note_covariant_thunk`.
  **L460 CN**: 声明 TableGen def 记录 `note_covariant_thunk`。
- **L461 EN**: Adds a standalone statement or declaration: `"covariant thunk required by %0">;`.
  **L461 CN**: 添加一条独立语句或声明：`"covariant thunk required by %0">;`。
- **L462 EN**: Closes the current lexical scope or compound statement.
  **L462 CN**: 结束当前词法作用域或复合语句块。
- **L463 EN**: Blank line separating nearby declarations or logic blocks.
  **L463 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L464 EN**: Comment explains nearby logic, constraints, or intent: `Importing ASTs`.
  **L464 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Importing ASTs`。
- **L465 EN**: Declares TableGen def record `err_odr_variable_type_inconsistent`.
  **L465 CN**: 声明 TableGen def 记录 `err_odr_variable_type_inconsistent`。
- **L466 EN**: Continues the surrounding expression or declaration: `"external variable %0 declared with incompatible types in different "`.
  **L466 CN**: 继续构造周围的表达式或声明：`"external variable %0 declared with incompatible types in different "`。
- **L467 EN**: Executes a call or declaration centered on `units`.
  **L467 CN**: 执行以 `units` 为核心的调用或声明。
- **L468 EN**: Declares TableGen def record `warn_odr_variable_type_inconsistent`.
  **L468 CN**: 声明 TableGen def 记录 `warn_odr_variable_type_inconsistent`。
- **L469 EN**: Continues the surrounding expression or declaration: `"external variable %0 declared with incompatible types in different "`.
  **L469 CN**: 继续构造周围的表达式或声明：`"external variable %0 declared with incompatible types in different "`。
- **L470 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"translation units (%1 vs. %2)">,`.
  **L470 CN**: 继续一个多行参数列表、初始化器或聚合项：`"translation units (%1 vs. %2)">,`。
- **L471 EN**: Adds a standalone statement or declaration: `InGroup<ODR>;`.
  **L471 CN**: 添加一条独立语句或声明：`InGroup<ODR>;`。
- **L472 EN**: Declares TableGen def record `err_odr_variable_multiple_def`.
  **L472 CN**: 声明 TableGen def 记录 `err_odr_variable_multiple_def`。
- **L473 EN**: Adds a standalone statement or declaration: `"external variable %0 defined in multiple translation units">;`.
  **L473 CN**: 添加一条独立语句或声明：`"external variable %0 defined in multiple translation units">;`。
- **L474 EN**: Declares TableGen def record `warn_odr_variable_multiple_def`.
  **L474 CN**: 声明 TableGen def 记录 `warn_odr_variable_multiple_def`。
- **L475 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"external variable %0 defined in multiple translation units">,`.
  **L475 CN**: 继续一个多行参数列表、初始化器或聚合项：`"external variable %0 defined in multiple translation units">,`。
- **L476 EN**: Adds a standalone statement or declaration: `InGroup<ODR>;`.
  **L476 CN**: 添加一条独立语句或声明：`InGroup<ODR>;`。
- **L477 EN**: Declares TableGen def record `note_odr_value_here`.
  **L477 CN**: 声明 TableGen def 记录 `note_odr_value_here`。
- **L478 EN**: Declares TableGen def record `err_odr_function_type_inconsistent`.
  **L478 CN**: 声明 TableGen def 记录 `err_odr_function_type_inconsistent`。
- **L479 EN**: Continues the surrounding expression or declaration: `"external function %0 declared with incompatible types in different "`.
  **L479 CN**: 继续构造周围的表达式或声明：`"external function %0 declared with incompatible types in different "`。
- **L480 EN**: Executes a call or declaration centered on `units`.
  **L480 CN**: 执行以 `units` 为核心的调用或声明。

### Lines 481-504

````tablegen
def warn_odr_function_type_inconsistent : Warning<
  "external function %0 declared with incompatible types in different "
  "translation units (%1 vs. %2)">,
  InGroup<ODR>;
def warn_odr_tag_type_with_attributes : Warning<
  "type %0 has %select{an attribute|a member with an attribute}1 which "
  "currently causes the types to be treated as though they are incompatible">,
  InGroup<ODR>, DefaultError;
def note_odr_attr_here : Note<"attribute %0 here">;
def err_odr_tag_type_inconsistent
    : Error<"type %0 has incompatible definitions%select{| in different "
            "translation units}1">;
def warn_odr_tag_type_inconsistent
    : Warning<"type %0 has incompatible definitions%select{| in different "
              "translation units}1">,
      InGroup<ODR>;
def note_odr_tag_kind_here: Note<
  "%0 is a %select{struct|interface|union|class|enum}1 here">;
def note_odr_field : Note<"field %0 has type %1 here">;
def note_odr_field_bit_width : Note<"bit-field %0 has bit-width %1 here">;
def note_odr_field_not_bit_field : Note<"field %0 is not a bit-field">;
def note_odr_field_name : Note<"field has name %0 here">;
def note_odr_missing_field : Note<"no corresponding field here">;
def note_odr_base : Note<"class has base type %0">;
````
- **L481 EN**: Declares TableGen def record `warn_odr_function_type_inconsistent`.
  **L481 CN**: 声明 TableGen def 记录 `warn_odr_function_type_inconsistent`。
- **L482 EN**: Continues the surrounding expression or declaration: `"external function %0 declared with incompatible types in different "`.
  **L482 CN**: 继续构造周围的表达式或声明：`"external function %0 declared with incompatible types in different "`。
- **L483 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"translation units (%1 vs. %2)">,`.
  **L483 CN**: 继续一个多行参数列表、初始化器或聚合项：`"translation units (%1 vs. %2)">,`。
- **L484 EN**: Adds a standalone statement or declaration: `InGroup<ODR>;`.
  **L484 CN**: 添加一条独立语句或声明：`InGroup<ODR>;`。
- **L485 EN**: Declares TableGen def record `warn_odr_tag_type_with_attributes`.
  **L485 CN**: 声明 TableGen def 记录 `warn_odr_tag_type_with_attributes`。
- **L486 EN**: Continues the surrounding expression or declaration: `"type %0 has %select{an attribute|a member with an attribute}1 which "`.
  **L486 CN**: 继续构造周围的表达式或声明：`"type %0 has %select{an attribute|a member with an attribute}1 which "`。
- **L487 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"currently causes the types to be treated as though they are incompatible">,`.
  **L487 CN**: 继续一个多行参数列表、初始化器或聚合项：`"currently causes the types to be treated as though they are incompatible">,`。
- **L488 EN**: Adds a standalone statement or declaration: `InGroup<ODR>, DefaultError;`.
  **L488 CN**: 添加一条独立语句或声明：`InGroup<ODR>, DefaultError;`。
- **L489 EN**: Declares TableGen def record `note_odr_attr_here`.
  **L489 CN**: 声明 TableGen def 记录 `note_odr_attr_here`。
- **L490 EN**: Declares TableGen def record `err_odr_tag_type_inconsistent`.
  **L490 CN**: 声明 TableGen def 记录 `err_odr_tag_type_inconsistent`。
- **L491 EN**: Continues the surrounding expression or declaration: `: Error<"type %0 has incompatible definitions%select{| in different "`.
  **L491 CN**: 继续构造周围的表达式或声明：`: Error<"type %0 has incompatible definitions%select{| in different "`。
- **L492 EN**: Adds a standalone statement or declaration: `"translation units}1">;`.
  **L492 CN**: 添加一条独立语句或声明：`"translation units}1">;`。
- **L493 EN**: Declares TableGen def record `warn_odr_tag_type_inconsistent`.
  **L493 CN**: 声明 TableGen def 记录 `warn_odr_tag_type_inconsistent`。
- **L494 EN**: Continues the surrounding expression or declaration: `: Warning<"type %0 has incompatible definitions%select{| in different "`.
  **L494 CN**: 继续构造周围的表达式或声明：`: Warning<"type %0 has incompatible definitions%select{| in different "`。
- **L495 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"translation units}1">,`.
  **L495 CN**: 继续一个多行参数列表、初始化器或聚合项：`"translation units}1">,`。
- **L496 EN**: Adds a standalone statement or declaration: `InGroup<ODR>;`.
  **L496 CN**: 添加一条独立语句或声明：`InGroup<ODR>;`。
- **L497 EN**: Declares TableGen def record `note_odr_tag_kind_here`.
  **L497 CN**: 声明 TableGen def 记录 `note_odr_tag_kind_here`。
- **L498 EN**: Adds a standalone statement or declaration: `"%0 is a %select{struct|interface|union|class|enum}1 here">;`.
  **L498 CN**: 添加一条独立语句或声明：`"%0 is a %select{struct|interface|union|class|enum}1 here">;`。
- **L499 EN**: Declares TableGen def record `note_odr_field`.
  **L499 CN**: 声明 TableGen def 记录 `note_odr_field`。
- **L500 EN**: Declares TableGen def record `note_odr_field_bit_width`.
  **L500 CN**: 声明 TableGen def 记录 `note_odr_field_bit_width`。
- **L501 EN**: Declares TableGen def record `note_odr_field_not_bit_field`.
  **L501 CN**: 声明 TableGen def 记录 `note_odr_field_not_bit_field`。
- **L502 EN**: Declares TableGen def record `note_odr_field_name`.
  **L502 CN**: 声明 TableGen def 记录 `note_odr_field_name`。
- **L503 EN**: Declares TableGen def record `note_odr_missing_field`.
  **L503 CN**: 声明 TableGen def 记录 `note_odr_missing_field`。
- **L504 EN**: Declares TableGen def record `note_odr_base`.
  **L504 CN**: 声明 TableGen def 记录 `note_odr_base`。

### Lines 505-528

````tablegen
def note_odr_virtual_base : Note<
  "%select{non-virtual|virtual}0 derivation here">;
def note_odr_missing_base : Note<"no corresponding base class here">;
def note_odr_number_of_bases : Note<
  "class has %0 base %plural{1:class|:classes}0">;
def note_odr_enumerator : Note<"enumerator %0 with value %1 here">;
def note_odr_missing_enumerator : Note<"no corresponding enumerator here">;
def note_odr_incompatible_fixed_underlying_type : Note<
  "enumeration %0 declared with incompatible fixed underlying types (%1 vs. "
  "%2)">;
def note_odr_fixed_underlying_type : Note<
  "enumeration %0 has fixed underlying type here">;
def note_odr_missing_fixed_underlying_type : Note<
  "enumeration %0 missing fixed underlying type here">;

def err_odr_field_type_inconsistent : Error<
  "field %0 declared with incompatible types in different "
  "translation units (%1 vs. %2)">;
def warn_odr_field_type_inconsistent : Warning<
  "field %0 declared with incompatible types in different "
  "translation units (%1 vs. %2)">,
  InGroup<ODR>;

// Importing Objective-C ASTs
````
- **L505 EN**: Declares TableGen def record `note_odr_virtual_base`.
  **L505 CN**: 声明 TableGen def 记录 `note_odr_virtual_base`。
- **L506 EN**: Adds a standalone statement or declaration: `"%select{non-virtual|virtual}0 derivation here">;`.
  **L506 CN**: 添加一条独立语句或声明：`"%select{non-virtual|virtual}0 derivation here">;`。
- **L507 EN**: Declares TableGen def record `note_odr_missing_base`.
  **L507 CN**: 声明 TableGen def 记录 `note_odr_missing_base`。
- **L508 EN**: Declares TableGen def record `note_odr_number_of_bases`.
  **L508 CN**: 声明 TableGen def 记录 `note_odr_number_of_bases`。
- **L509 EN**: Adds a standalone statement or declaration: `"class has %0 base %plural{1:class|:classes}0">;`.
  **L509 CN**: 添加一条独立语句或声明：`"class has %0 base %plural{1:class|:classes}0">;`。
- **L510 EN**: Declares TableGen def record `note_odr_enumerator`.
  **L510 CN**: 声明 TableGen def 记录 `note_odr_enumerator`。
- **L511 EN**: Declares TableGen def record `note_odr_missing_enumerator`.
  **L511 CN**: 声明 TableGen def 记录 `note_odr_missing_enumerator`。
- **L512 EN**: Declares TableGen def record `note_odr_incompatible_fixed_underlying_type`.
  **L512 CN**: 声明 TableGen def 记录 `note_odr_incompatible_fixed_underlying_type`。
- **L513 EN**: Continues logic associated with callable symbol `types`.
  **L513 CN**: 继续与可调用符号 `types` 相关的逻辑。
- **L514 EN**: Adds a standalone statement or declaration: `"%2)">;`.
  **L514 CN**: 添加一条独立语句或声明：`"%2)">;`。
- **L515 EN**: Declares TableGen def record `note_odr_fixed_underlying_type`.
  **L515 CN**: 声明 TableGen def 记录 `note_odr_fixed_underlying_type`。
- **L516 EN**: Adds a standalone statement or declaration: `"enumeration %0 has fixed underlying type here">;`.
  **L516 CN**: 添加一条独立语句或声明：`"enumeration %0 has fixed underlying type here">;`。
- **L517 EN**: Declares TableGen def record `note_odr_missing_fixed_underlying_type`.
  **L517 CN**: 声明 TableGen def 记录 `note_odr_missing_fixed_underlying_type`。
- **L518 EN**: Adds a standalone statement or declaration: `"enumeration %0 missing fixed underlying type here">;`.
  **L518 CN**: 添加一条独立语句或声明：`"enumeration %0 missing fixed underlying type here">;`。
- **L519 EN**: Blank line separating nearby declarations or logic blocks.
  **L519 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L520 EN**: Declares TableGen def record `err_odr_field_type_inconsistent`.
  **L520 CN**: 声明 TableGen def 记录 `err_odr_field_type_inconsistent`。
- **L521 EN**: Continues the surrounding expression or declaration: `"field %0 declared with incompatible types in different "`.
  **L521 CN**: 继续构造周围的表达式或声明：`"field %0 declared with incompatible types in different "`。
- **L522 EN**: Executes a call or declaration centered on `units`.
  **L522 CN**: 执行以 `units` 为核心的调用或声明。
- **L523 EN**: Declares TableGen def record `warn_odr_field_type_inconsistent`.
  **L523 CN**: 声明 TableGen def 记录 `warn_odr_field_type_inconsistent`。
- **L524 EN**: Continues the surrounding expression or declaration: `"field %0 declared with incompatible types in different "`.
  **L524 CN**: 继续构造周围的表达式或声明：`"field %0 declared with incompatible types in different "`。
- **L525 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"translation units (%1 vs. %2)">,`.
  **L525 CN**: 继续一个多行参数列表、初始化器或聚合项：`"translation units (%1 vs. %2)">,`。
- **L526 EN**: Adds a standalone statement or declaration: `InGroup<ODR>;`.
  **L526 CN**: 添加一条独立语句或声明：`InGroup<ODR>;`。
- **L527 EN**: Blank line separating nearby declarations or logic blocks.
  **L527 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L528 EN**: Comment explains nearby logic, constraints, or intent: `Importing Objective-C ASTs`.
  **L528 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Importing Objective-C ASTs`。

### Lines 529-552

````tablegen
def err_odr_ivar_type_inconsistent : Error<
  "instance variable %0 declared with incompatible types in different "
  "translation units (%1 vs. %2)">;
def warn_odr_ivar_type_inconsistent : Warning<
  "instance variable %0 declared with incompatible types in different "
  "translation units (%1 vs. %2)">,
  InGroup<ODR>;
def err_odr_objc_superclass_inconsistent : Error<
  "class %0 has incompatible superclasses">;
def warn_odr_objc_superclass_inconsistent : Warning<
  "class %0 has incompatible superclasses">,
  InGroup<ODR>;
def note_odr_objc_superclass : Note<"inherits from superclass %0 here">;
def note_odr_objc_missing_superclass : Note<"no corresponding superclass here">;
def err_odr_objc_method_result_type_inconsistent : Error<
  "%select{class|instance}0 method %1 has incompatible result types in "
  "different translation units (%2 vs. %3)">;
def warn_odr_objc_method_result_type_inconsistent : Warning<
  "%select{class|instance}0 method %1 has incompatible result types in "
  "different translation units (%2 vs. %3)">,
  InGroup<ODR>;
def err_odr_objc_method_num_params_inconsistent : Error<
  "%select{class|instance}0 method %1 has a different number of parameters in "
  "different translation units (%2 vs. %3)">;
````
- **L529 EN**: Declares TableGen def record `err_odr_ivar_type_inconsistent`.
  **L529 CN**: 声明 TableGen def 记录 `err_odr_ivar_type_inconsistent`。
- **L530 EN**: Continues the surrounding expression or declaration: `"instance variable %0 declared with incompatible types in different "`.
  **L530 CN**: 继续构造周围的表达式或声明：`"instance variable %0 declared with incompatible types in different "`。
- **L531 EN**: Executes a call or declaration centered on `units`.
  **L531 CN**: 执行以 `units` 为核心的调用或声明。
- **L532 EN**: Declares TableGen def record `warn_odr_ivar_type_inconsistent`.
  **L532 CN**: 声明 TableGen def 记录 `warn_odr_ivar_type_inconsistent`。
- **L533 EN**: Continues the surrounding expression or declaration: `"instance variable %0 declared with incompatible types in different "`.
  **L533 CN**: 继续构造周围的表达式或声明：`"instance variable %0 declared with incompatible types in different "`。
- **L534 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"translation units (%1 vs. %2)">,`.
  **L534 CN**: 继续一个多行参数列表、初始化器或聚合项：`"translation units (%1 vs. %2)">,`。
- **L535 EN**: Adds a standalone statement or declaration: `InGroup<ODR>;`.
  **L535 CN**: 添加一条独立语句或声明：`InGroup<ODR>;`。
- **L536 EN**: Declares TableGen def record `err_odr_objc_superclass_inconsistent`.
  **L536 CN**: 声明 TableGen def 记录 `err_odr_objc_superclass_inconsistent`。
- **L537 EN**: Adds a standalone statement or declaration: `"class %0 has incompatible superclasses">;`.
  **L537 CN**: 添加一条独立语句或声明：`"class %0 has incompatible superclasses">;`。
- **L538 EN**: Declares TableGen def record `warn_odr_objc_superclass_inconsistent`.
  **L538 CN**: 声明 TableGen def 记录 `warn_odr_objc_superclass_inconsistent`。
- **L539 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"class %0 has incompatible superclasses">,`.
  **L539 CN**: 继续一个多行参数列表、初始化器或聚合项：`"class %0 has incompatible superclasses">,`。
- **L540 EN**: Adds a standalone statement or declaration: `InGroup<ODR>;`.
  **L540 CN**: 添加一条独立语句或声明：`InGroup<ODR>;`。
- **L541 EN**: Declares TableGen def record `note_odr_objc_superclass`.
  **L541 CN**: 声明 TableGen def 记录 `note_odr_objc_superclass`。
- **L542 EN**: Declares TableGen def record `note_odr_objc_missing_superclass`.
  **L542 CN**: 声明 TableGen def 记录 `note_odr_objc_missing_superclass`。
- **L543 EN**: Declares TableGen def record `err_odr_objc_method_result_type_inconsistent`.
  **L543 CN**: 声明 TableGen def 记录 `err_odr_objc_method_result_type_inconsistent`。
- **L544 EN**: Continues the surrounding expression or declaration: `"%select{class|instance}0 method %1 has incompatible result types in "`.
  **L544 CN**: 继续构造周围的表达式或声明：`"%select{class|instance}0 method %1 has incompatible result types in "`。
- **L545 EN**: Executes a call or declaration centered on `units`.
  **L545 CN**: 执行以 `units` 为核心的调用或声明。
- **L546 EN**: Declares TableGen def record `warn_odr_objc_method_result_type_inconsistent`.
  **L546 CN**: 声明 TableGen def 记录 `warn_odr_objc_method_result_type_inconsistent`。
- **L547 EN**: Continues the surrounding expression or declaration: `"%select{class|instance}0 method %1 has incompatible result types in "`.
  **L547 CN**: 继续构造周围的表达式或声明：`"%select{class|instance}0 method %1 has incompatible result types in "`。
- **L548 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"different translation units (%2 vs. %3)">,`.
  **L548 CN**: 继续一个多行参数列表、初始化器或聚合项：`"different translation units (%2 vs. %3)">,`。
- **L549 EN**: Adds a standalone statement or declaration: `InGroup<ODR>;`.
  **L549 CN**: 添加一条独立语句或声明：`InGroup<ODR>;`。
- **L550 EN**: Declares TableGen def record `err_odr_objc_method_num_params_inconsistent`.
  **L550 CN**: 声明 TableGen def 记录 `err_odr_objc_method_num_params_inconsistent`。
- **L551 EN**: Continues the surrounding expression or declaration: `"%select{class|instance}0 method %1 has a different number of parameters in "`.
  **L551 CN**: 继续构造周围的表达式或声明：`"%select{class|instance}0 method %1 has a different number of parameters in "`。
- **L552 EN**: Executes a call or declaration centered on `units`.
  **L552 CN**: 执行以 `units` 为核心的调用或声明。

### Lines 553-576

````tablegen
def warn_odr_objc_method_num_params_inconsistent : Warning<
  "%select{class|instance}0 method %1 has a different number of parameters in "
  "different translation units (%2 vs. %3)">,
  InGroup<ODR>;
def err_odr_objc_method_param_type_inconsistent : Error<
  "%select{class|instance}0 method %1 has a parameter with a different types "
  "in different translation units (%2 vs. %3)">;
def warn_odr_objc_method_param_type_inconsistent : Warning<
  "%select{class|instance}0 method %1 has a parameter with a different types "
  "in different translation units (%2 vs. %3)">,
  InGroup<ODR>;
def err_odr_objc_method_variadic_inconsistent : Error<
  "%select{class|instance}0 method %1 is variadic in one translation unit "
  "and not variadic in another">;
def warn_odr_objc_method_variadic_inconsistent : Warning<
  "%select{class|instance}0 method %1 is variadic in one translation unit "
  "and not variadic in another">,
  InGroup<ODR>;
def note_odr_objc_method_here : Note<
  "%select{class|instance}0 method %1 also declared here">;
def err_odr_objc_property_type_inconsistent : Error<
  "property %0 declared with incompatible types in different "
  "translation units (%1 vs. %2)">;
def warn_odr_objc_property_type_inconsistent : Warning<
````
- **L553 EN**: Declares TableGen def record `warn_odr_objc_method_num_params_inconsistent`.
  **L553 CN**: 声明 TableGen def 记录 `warn_odr_objc_method_num_params_inconsistent`。
- **L554 EN**: Continues the surrounding expression or declaration: `"%select{class|instance}0 method %1 has a different number of parameters in "`.
  **L554 CN**: 继续构造周围的表达式或声明：`"%select{class|instance}0 method %1 has a different number of parameters in "`。
- **L555 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"different translation units (%2 vs. %3)">,`.
  **L555 CN**: 继续一个多行参数列表、初始化器或聚合项：`"different translation units (%2 vs. %3)">,`。
- **L556 EN**: Adds a standalone statement or declaration: `InGroup<ODR>;`.
  **L556 CN**: 添加一条独立语句或声明：`InGroup<ODR>;`。
- **L557 EN**: Declares TableGen def record `err_odr_objc_method_param_type_inconsistent`.
  **L557 CN**: 声明 TableGen def 记录 `err_odr_objc_method_param_type_inconsistent`。
- **L558 EN**: Continues the surrounding expression or declaration: `"%select{class|instance}0 method %1 has a parameter with a different types "`.
  **L558 CN**: 继续构造周围的表达式或声明：`"%select{class|instance}0 method %1 has a parameter with a different types "`。
- **L559 EN**: Executes a call or declaration centered on `units`.
  **L559 CN**: 执行以 `units` 为核心的调用或声明。
- **L560 EN**: Declares TableGen def record `warn_odr_objc_method_param_type_inconsistent`.
  **L560 CN**: 声明 TableGen def 记录 `warn_odr_objc_method_param_type_inconsistent`。
- **L561 EN**: Continues the surrounding expression or declaration: `"%select{class|instance}0 method %1 has a parameter with a different types "`.
  **L561 CN**: 继续构造周围的表达式或声明：`"%select{class|instance}0 method %1 has a parameter with a different types "`。
- **L562 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"in different translation units (%2 vs. %3)">,`.
  **L562 CN**: 继续一个多行参数列表、初始化器或聚合项：`"in different translation units (%2 vs. %3)">,`。
- **L563 EN**: Adds a standalone statement or declaration: `InGroup<ODR>;`.
  **L563 CN**: 添加一条独立语句或声明：`InGroup<ODR>;`。
- **L564 EN**: Declares TableGen def record `err_odr_objc_method_variadic_inconsistent`.
  **L564 CN**: 声明 TableGen def 记录 `err_odr_objc_method_variadic_inconsistent`。
- **L565 EN**: Continues the surrounding expression or declaration: `"%select{class|instance}0 method %1 is variadic in one translation unit "`.
  **L565 CN**: 继续构造周围的表达式或声明：`"%select{class|instance}0 method %1 is variadic in one translation unit "`。
- **L566 EN**: Adds a standalone statement or declaration: `"and not variadic in another">;`.
  **L566 CN**: 添加一条独立语句或声明：`"and not variadic in another">;`。
- **L567 EN**: Declares TableGen def record `warn_odr_objc_method_variadic_inconsistent`.
  **L567 CN**: 声明 TableGen def 记录 `warn_odr_objc_method_variadic_inconsistent`。
- **L568 EN**: Continues the surrounding expression or declaration: `"%select{class|instance}0 method %1 is variadic in one translation unit "`.
  **L568 CN**: 继续构造周围的表达式或声明：`"%select{class|instance}0 method %1 is variadic in one translation unit "`。
- **L569 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"and not variadic in another">,`.
  **L569 CN**: 继续一个多行参数列表、初始化器或聚合项：`"and not variadic in another">,`。
- **L570 EN**: Adds a standalone statement or declaration: `InGroup<ODR>;`.
  **L570 CN**: 添加一条独立语句或声明：`InGroup<ODR>;`。
- **L571 EN**: Declares TableGen def record `note_odr_objc_method_here`.
  **L571 CN**: 声明 TableGen def 记录 `note_odr_objc_method_here`。
- **L572 EN**: Adds a standalone statement or declaration: `"%select{class|instance}0 method %1 also declared here">;`.
  **L572 CN**: 添加一条独立语句或声明：`"%select{class|instance}0 method %1 also declared here">;`。
- **L573 EN**: Declares TableGen def record `err_odr_objc_property_type_inconsistent`.
  **L573 CN**: 声明 TableGen def 记录 `err_odr_objc_property_type_inconsistent`。
- **L574 EN**: Continues the surrounding expression or declaration: `"property %0 declared with incompatible types in different "`.
  **L574 CN**: 继续构造周围的表达式或声明：`"property %0 declared with incompatible types in different "`。
- **L575 EN**: Executes a call or declaration centered on `units`.
  **L575 CN**: 执行以 `units` 为核心的调用或声明。
- **L576 EN**: Declares TableGen def record `warn_odr_objc_property_type_inconsistent`.
  **L576 CN**: 声明 TableGen def 记录 `warn_odr_objc_property_type_inconsistent`。

### Lines 577-600

````tablegen
  "property %0 declared with incompatible types in different "
  "translation units (%1 vs. %2)">,
  InGroup<ODR>;
def err_odr_objc_property_impl_kind_inconsistent : Error<
  "property %0 is implemented with %select{@synthesize|@dynamic}1 in one "
  "translation but %select{@dynamic|@synthesize}1 in another translation unit">;
def warn_odr_objc_property_impl_kind_inconsistent : Warning<
  "property %0 is implemented with %select{@synthesize|@dynamic}1 in one "
  "translation but %select{@dynamic|@synthesize}1 in another translation unit">,
  InGroup<ODR>;
def note_odr_objc_property_impl_kind : Note<
  "property %0 is implemented with %select{@synthesize|@dynamic}1 here">;
def err_odr_objc_synthesize_ivar_inconsistent : Error<
  "property %0 is synthesized to different ivars in different translation "
  "units (%1 vs. %2)">;
def warn_odr_objc_synthesize_ivar_inconsistent : Warning<
  "property %0 is synthesized to different ivars in different translation "
  "units (%1 vs. %2)">,
  InGroup<ODR>;
def note_odr_objc_synthesize_ivar_here : Note<
  "property is synthesized to ivar %0 here">;
def err_unsupported_objc_primitive_encoding
    : Error<"cannot yet @encode type %0">;

````
- **L577 EN**: Continues the surrounding expression or declaration: `"property %0 declared with incompatible types in different "`.
  **L577 CN**: 继续构造周围的表达式或声明：`"property %0 declared with incompatible types in different "`。
- **L578 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"translation units (%1 vs. %2)">,`.
  **L578 CN**: 继续一个多行参数列表、初始化器或聚合项：`"translation units (%1 vs. %2)">,`。
- **L579 EN**: Adds a standalone statement or declaration: `InGroup<ODR>;`.
  **L579 CN**: 添加一条独立语句或声明：`InGroup<ODR>;`。
- **L580 EN**: Declares TableGen def record `err_odr_objc_property_impl_kind_inconsistent`.
  **L580 CN**: 声明 TableGen def 记录 `err_odr_objc_property_impl_kind_inconsistent`。
- **L581 EN**: Continues the surrounding expression or declaration: `"property %0 is implemented with %select{@synthesize|@dynamic}1 in one "`.
  **L581 CN**: 继续构造周围的表达式或声明：`"property %0 is implemented with %select{@synthesize|@dynamic}1 in one "`。
- **L582 EN**: Adds a standalone statement or declaration: `"translation but %select{@dynamic|@synthesize}1 in another translation unit">;`.
  **L582 CN**: 添加一条独立语句或声明：`"translation but %select{@dynamic|@synthesize}1 in another translation unit">;`。
- **L583 EN**: Declares TableGen def record `warn_odr_objc_property_impl_kind_inconsistent`.
  **L583 CN**: 声明 TableGen def 记录 `warn_odr_objc_property_impl_kind_inconsistent`。
- **L584 EN**: Continues the surrounding expression or declaration: `"property %0 is implemented with %select{@synthesize|@dynamic}1 in one "`.
  **L584 CN**: 继续构造周围的表达式或声明：`"property %0 is implemented with %select{@synthesize|@dynamic}1 in one "`。
- **L585 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"translation but %select{@dynamic|@synthesize}1 in another translation unit">,`.
  **L585 CN**: 继续一个多行参数列表、初始化器或聚合项：`"translation but %select{@dynamic|@synthesize}1 in another translation unit">,`。
- **L586 EN**: Adds a standalone statement or declaration: `InGroup<ODR>;`.
  **L586 CN**: 添加一条独立语句或声明：`InGroup<ODR>;`。
- **L587 EN**: Declares TableGen def record `note_odr_objc_property_impl_kind`.
  **L587 CN**: 声明 TableGen def 记录 `note_odr_objc_property_impl_kind`。
- **L588 EN**: Adds a standalone statement or declaration: `"property %0 is implemented with %select{@synthesize|@dynamic}1 here">;`.
  **L588 CN**: 添加一条独立语句或声明：`"property %0 is implemented with %select{@synthesize|@dynamic}1 here">;`。
- **L589 EN**: Declares TableGen def record `err_odr_objc_synthesize_ivar_inconsistent`.
  **L589 CN**: 声明 TableGen def 记录 `err_odr_objc_synthesize_ivar_inconsistent`。
- **L590 EN**: Continues the surrounding expression or declaration: `"property %0 is synthesized to different ivars in different translation "`.
  **L590 CN**: 继续构造周围的表达式或声明：`"property %0 is synthesized to different ivars in different translation "`。
- **L591 EN**: Executes a call or declaration centered on `"units`.
  **L591 CN**: 执行以 `"units` 为核心的调用或声明。
- **L592 EN**: Declares TableGen def record `warn_odr_objc_synthesize_ivar_inconsistent`.
  **L592 CN**: 声明 TableGen def 记录 `warn_odr_objc_synthesize_ivar_inconsistent`。
- **L593 EN**: Continues the surrounding expression or declaration: `"property %0 is synthesized to different ivars in different translation "`.
  **L593 CN**: 继续构造周围的表达式或声明：`"property %0 is synthesized to different ivars in different translation "`。
- **L594 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"units (%1 vs. %2)">,`.
  **L594 CN**: 继续一个多行参数列表、初始化器或聚合项：`"units (%1 vs. %2)">,`。
- **L595 EN**: Adds a standalone statement or declaration: `InGroup<ODR>;`.
  **L595 CN**: 添加一条独立语句或声明：`InGroup<ODR>;`。
- **L596 EN**: Declares TableGen def record `note_odr_objc_synthesize_ivar_here`.
  **L596 CN**: 声明 TableGen def 记录 `note_odr_objc_synthesize_ivar_here`。
- **L597 EN**: Adds a standalone statement or declaration: `"property is synthesized to ivar %0 here">;`.
  **L597 CN**: 添加一条独立语句或声明：`"property is synthesized to ivar %0 here">;`。
- **L598 EN**: Declares TableGen def record `err_unsupported_objc_primitive_encoding`.
  **L598 CN**: 声明 TableGen def 记录 `err_unsupported_objc_primitive_encoding`。
- **L599 EN**: Adds a standalone statement or declaration: `: Error<"cannot yet @encode type %0">;`.
  **L599 CN**: 添加一条独立语句或声明：`: Error<"cannot yet @encode type %0">;`。
- **L600 EN**: Blank line separating nearby declarations or logic blocks.
  **L600 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 601-624

````tablegen
// Importing C++ ASTs
def note_odr_friend : Note<"friend declared here">;
def note_odr_missing_friend : Note<"no corresponding friend here">;
def err_odr_different_num_template_parameters : Error<
  "template parameter lists have a different number of parameters (%0 vs %1)">;
def warn_odr_different_num_template_parameters : Warning<
  "template parameter lists have a different number of parameters (%0 vs %1)">,
  InGroup<ODR>;
def note_odr_template_parameter_list : Note<
  "template parameter list also declared here">;
def err_odr_different_template_parameter_kind : Error<
  "template parameter has different kinds in different translation units">;
def warn_odr_different_template_parameter_kind : Warning<
  "template parameter has different kinds in different translation units">,
  InGroup<ODR>;
def note_odr_template_parameter_here : Note<
  "template parameter declared here">;
def err_odr_parameter_pack_non_pack : Error<
  "parameter kind mismatch; parameter is %select{not a|a}0 parameter pack">;
def warn_odr_parameter_pack_non_pack : Warning<
  "parameter kind mismatch; parameter is %select{not a|a}0 parameter pack">,
  InGroup<ODR>;
def note_odr_parameter_pack_non_pack : Note<
  "%select{parameter|parameter pack}0 declared here">;
````
- **L601 EN**: Comment explains nearby logic, constraints, or intent: `Importing C++ ASTs`.
  **L601 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Importing C++ ASTs`。
- **L602 EN**: Declares TableGen def record `note_odr_friend`.
  **L602 CN**: 声明 TableGen def 记录 `note_odr_friend`。
- **L603 EN**: Declares TableGen def record `note_odr_missing_friend`.
  **L603 CN**: 声明 TableGen def 记录 `note_odr_missing_friend`。
- **L604 EN**: Declares TableGen def record `err_odr_different_num_template_parameters`.
  **L604 CN**: 声明 TableGen def 记录 `err_odr_different_num_template_parameters`。
- **L605 EN**: Executes a call or declaration centered on `parameters`.
  **L605 CN**: 执行以 `parameters` 为核心的调用或声明。
- **L606 EN**: Declares TableGen def record `warn_odr_different_num_template_parameters`.
  **L606 CN**: 声明 TableGen def 记录 `warn_odr_different_num_template_parameters`。
- **L607 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"template parameter lists have a different number of parameters (%0 vs %1)">,`.
  **L607 CN**: 继续一个多行参数列表、初始化器或聚合项：`"template parameter lists have a different number of parameters (%0 vs %1)">,`。
- **L608 EN**: Adds a standalone statement or declaration: `InGroup<ODR>;`.
  **L608 CN**: 添加一条独立语句或声明：`InGroup<ODR>;`。
- **L609 EN**: Declares TableGen def record `note_odr_template_parameter_list`.
  **L609 CN**: 声明 TableGen def 记录 `note_odr_template_parameter_list`。
- **L610 EN**: Adds a standalone statement or declaration: `"template parameter list also declared here">;`.
  **L610 CN**: 添加一条独立语句或声明：`"template parameter list also declared here">;`。
- **L611 EN**: Declares TableGen def record `err_odr_different_template_parameter_kind`.
  **L611 CN**: 声明 TableGen def 记录 `err_odr_different_template_parameter_kind`。
- **L612 EN**: Adds a standalone statement or declaration: `"template parameter has different kinds in different translation units">;`.
  **L612 CN**: 添加一条独立语句或声明：`"template parameter has different kinds in different translation units">;`。
- **L613 EN**: Declares TableGen def record `warn_odr_different_template_parameter_kind`.
  **L613 CN**: 声明 TableGen def 记录 `warn_odr_different_template_parameter_kind`。
- **L614 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"template parameter has different kinds in different translation units">,`.
  **L614 CN**: 继续一个多行参数列表、初始化器或聚合项：`"template parameter has different kinds in different translation units">,`。
- **L615 EN**: Adds a standalone statement or declaration: `InGroup<ODR>;`.
  **L615 CN**: 添加一条独立语句或声明：`InGroup<ODR>;`。
- **L616 EN**: Declares TableGen def record `note_odr_template_parameter_here`.
  **L616 CN**: 声明 TableGen def 记录 `note_odr_template_parameter_here`。
- **L617 EN**: Adds a standalone statement or declaration: `"template parameter declared here">;`.
  **L617 CN**: 添加一条独立语句或声明：`"template parameter declared here">;`。
- **L618 EN**: Declares TableGen def record `err_odr_parameter_pack_non_pack`.
  **L618 CN**: 声明 TableGen def 记录 `err_odr_parameter_pack_non_pack`。
- **L619 EN**: Adds a standalone statement or declaration: `"parameter kind mismatch; parameter is %select{not a|a}0 parameter pack">;`.
  **L619 CN**: 添加一条独立语句或声明：`"parameter kind mismatch; parameter is %select{not a|a}0 parameter pack">;`。
- **L620 EN**: Declares TableGen def record `warn_odr_parameter_pack_non_pack`.
  **L620 CN**: 声明 TableGen def 记录 `warn_odr_parameter_pack_non_pack`。
- **L621 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"parameter kind mismatch; parameter is %select{not a|a}0 parameter pack">,`.
  **L621 CN**: 继续一个多行参数列表、初始化器或聚合项：`"parameter kind mismatch; parameter is %select{not a|a}0 parameter pack">,`。
- **L622 EN**: Adds a standalone statement or declaration: `InGroup<ODR>;`.
  **L622 CN**: 添加一条独立语句或声明：`InGroup<ODR>;`。
- **L623 EN**: Declares TableGen def record `note_odr_parameter_pack_non_pack`.
  **L623 CN**: 声明 TableGen def 记录 `note_odr_parameter_pack_non_pack`。
- **L624 EN**: Adds a standalone statement or declaration: `"%select{parameter|parameter pack}0 declared here">;`.
  **L624 CN**: 添加一条独立语句或声明：`"%select{parameter|parameter pack}0 declared here">;`。

### Lines 625-648

````tablegen
def err_odr_non_type_parameter_type_inconsistent : Error<
  "non-type template parameter declared with incompatible types in different "
  "translation units (%0 vs. %1)">;
def warn_odr_non_type_parameter_type_inconsistent : Warning<
  "non-type template parameter declared with incompatible types in different "
  "translation units (%0 vs. %1)">,
  InGroup<ODR>;
def err_unsupported_ast_node: Error<"cannot import unsupported AST node %0">;

// Compare ODR hashes
def err_module_odr_violation_different_definitions : Error<
  "%q0 has different definitions in different modules; "
  "%select{definition in module '%2' is here|defined here}1">;
def note_first_module_difference : Note<
  "in first definition, possible difference is here">;
def note_module_odr_violation_different_definitions : Note<
  "definition in module '%0' is here">;
def note_second_module_difference : Note<
  "in second definition, possible difference is here">;

def err_module_odr_violation_definition_data : Error <
  "%q0 has different definitions in different modules; first difference is "
  "%select{definition in module '%2'|defined here}1 found "
  "%select{"
````
- **L625 EN**: Declares TableGen def record `err_odr_non_type_parameter_type_inconsistent`.
  **L625 CN**: 声明 TableGen def 记录 `err_odr_non_type_parameter_type_inconsistent`。
- **L626 EN**: Continues the surrounding expression or declaration: `"non-type template parameter declared with incompatible types in different "`.
  **L626 CN**: 继续构造周围的表达式或声明：`"non-type template parameter declared with incompatible types in different "`。
- **L627 EN**: Executes a call or declaration centered on `units`.
  **L627 CN**: 执行以 `units` 为核心的调用或声明。
- **L628 EN**: Declares TableGen def record `warn_odr_non_type_parameter_type_inconsistent`.
  **L628 CN**: 声明 TableGen def 记录 `warn_odr_non_type_parameter_type_inconsistent`。
- **L629 EN**: Continues the surrounding expression or declaration: `"non-type template parameter declared with incompatible types in different "`.
  **L629 CN**: 继续构造周围的表达式或声明：`"non-type template parameter declared with incompatible types in different "`。
- **L630 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"translation units (%0 vs. %1)">,`.
  **L630 CN**: 继续一个多行参数列表、初始化器或聚合项：`"translation units (%0 vs. %1)">,`。
- **L631 EN**: Adds a standalone statement or declaration: `InGroup<ODR>;`.
  **L631 CN**: 添加一条独立语句或声明：`InGroup<ODR>;`。
- **L632 EN**: Declares TableGen def record `err_unsupported_ast_node`.
  **L632 CN**: 声明 TableGen def 记录 `err_unsupported_ast_node`。
- **L633 EN**: Blank line separating nearby declarations or logic blocks.
  **L633 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L634 EN**: Comment explains nearby logic, constraints, or intent: `Compare ODR hashes`.
  **L634 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Compare ODR hashes`。
- **L635 EN**: Declares TableGen def record `err_module_odr_violation_different_definitions`.
  **L635 CN**: 声明 TableGen def 记录 `err_module_odr_violation_different_definitions`。
- **L636 EN**: Continues the surrounding expression or declaration: `"%q0 has different definitions in different modules; "`.
  **L636 CN**: 继续构造周围的表达式或声明：`"%q0 has different definitions in different modules; "`。
- **L637 EN**: Adds a standalone statement or declaration: `"%select{definition in module '%2' is here|defined here}1">;`.
  **L637 CN**: 添加一条独立语句或声明：`"%select{definition in module '%2' is here|defined here}1">;`。
- **L638 EN**: Declares TableGen def record `note_first_module_difference`.
  **L638 CN**: 声明 TableGen def 记录 `note_first_module_difference`。
- **L639 EN**: Adds a standalone statement or declaration: `"in first definition, possible difference is here">;`.
  **L639 CN**: 添加一条独立语句或声明：`"in first definition, possible difference is here">;`。
- **L640 EN**: Declares TableGen def record `note_module_odr_violation_different_definitions`.
  **L640 CN**: 声明 TableGen def 记录 `note_module_odr_violation_different_definitions`。
- **L641 EN**: Adds a standalone statement or declaration: `"definition in module '%0' is here">;`.
  **L641 CN**: 添加一条独立语句或声明：`"definition in module '%0' is here">;`。
- **L642 EN**: Declares TableGen def record `note_second_module_difference`.
  **L642 CN**: 声明 TableGen def 记录 `note_second_module_difference`。
- **L643 EN**: Adds a standalone statement or declaration: `"in second definition, possible difference is here">;`.
  **L643 CN**: 添加一条独立语句或声明：`"in second definition, possible difference is here">;`。
- **L644 EN**: Blank line separating nearby declarations or logic blocks.
  **L644 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L645 EN**: Declares TableGen def record `err_module_odr_violation_definition_data`.
  **L645 CN**: 声明 TableGen def 记录 `err_module_odr_violation_definition_data`。
- **L646 EN**: Continues the surrounding expression or declaration: `"%q0 has different definitions in different modules; first difference is "`.
  **L646 CN**: 继续构造周围的表达式或声明：`"%q0 has different definitions in different modules; first difference is "`。
- **L647 EN**: Continues the surrounding expression or declaration: `"%select{definition in module '%2'|defined here}1 found "`.
  **L647 CN**: 继续构造周围的表达式或声明：`"%select{definition in module '%2'|defined here}1 found "`。
- **L648 EN**: Continues the surrounding expression or declaration: `"%select{"`.
  **L648 CN**: 继续构造周围的表达式或声明：`"%select{"`。

### Lines 649-672

````tablegen
  "%4 base %plural{1:class|:classes}4|"
  "%4 virtual base %plural{1:class|:classes}4|"
  "%ordinal4 base class with type %5|"
  "%ordinal4 %select{non-virtual|virtual}5 base class %6|"
  "%ordinal4 base class %5 with "
  "%select{public|protected|private|no}6 access specifier}3">;

def note_module_odr_violation_definition_data : Note <
  "but in '%0' found "
  "%select{"
  "%2 base %plural{1:class|:classes}2|"
  "%2 virtual base %plural{1:class|:classes}2|"
  "%ordinal2 base class with different type %3|"
  "%ordinal2 %select{non-virtual|virtual}3 base class %4|"
  "%ordinal2 base class %3 with "
  "%select{public|protected|private|no}4 access specifier}1">;

def err_module_odr_violation_objc_interface : Error <
  "%0 has different definitions in different modules; first difference is "
  "%select{definition in module '%2'|defined here}1 found "
  "%select{"
  "%select{no super class|super class with type %5}4|"
  "instance variable '%4' access control is "
    "%select{|@private|@protected|@public|@package}5"
````
- **L649 EN**: Continues the surrounding expression or declaration: `"%4 base %plural{1:class|:classes}4|"`.
  **L649 CN**: 继续构造周围的表达式或声明：`"%4 base %plural{1:class|:classes}4|"`。
- **L650 EN**: Continues the surrounding expression or declaration: `"%4 virtual base %plural{1:class|:classes}4|"`.
  **L650 CN**: 继续构造周围的表达式或声明：`"%4 virtual base %plural{1:class|:classes}4|"`。
- **L651 EN**: Continues the surrounding expression or declaration: `"%ordinal4 base class with type %5|"`.
  **L651 CN**: 继续构造周围的表达式或声明：`"%ordinal4 base class with type %5|"`。
- **L652 EN**: Continues the surrounding expression or declaration: `"%ordinal4 %select{non-virtual|virtual}5 base class %6|"`.
  **L652 CN**: 继续构造周围的表达式或声明：`"%ordinal4 %select{non-virtual|virtual}5 base class %6|"`。
- **L653 EN**: Continues the surrounding expression or declaration: `"%ordinal4 base class %5 with "`.
  **L653 CN**: 继续构造周围的表达式或声明：`"%ordinal4 base class %5 with "`。
- **L654 EN**: Adds a standalone statement or declaration: `"%select{public|protected|private|no}6 access specifier}3">;`.
  **L654 CN**: 添加一条独立语句或声明：`"%select{public|protected|private|no}6 access specifier}3">;`。
- **L655 EN**: Blank line separating nearby declarations or logic blocks.
  **L655 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L656 EN**: Declares TableGen def record `note_module_odr_violation_definition_data`.
  **L656 CN**: 声明 TableGen def 记录 `note_module_odr_violation_definition_data`。
- **L657 EN**: Continues the surrounding expression or declaration: `"but in '%0' found "`.
  **L657 CN**: 继续构造周围的表达式或声明：`"but in '%0' found "`。
- **L658 EN**: Continues the surrounding expression or declaration: `"%select{"`.
  **L658 CN**: 继续构造周围的表达式或声明：`"%select{"`。
- **L659 EN**: Continues the surrounding expression or declaration: `"%2 base %plural{1:class|:classes}2|"`.
  **L659 CN**: 继续构造周围的表达式或声明：`"%2 base %plural{1:class|:classes}2|"`。
- **L660 EN**: Continues the surrounding expression or declaration: `"%2 virtual base %plural{1:class|:classes}2|"`.
  **L660 CN**: 继续构造周围的表达式或声明：`"%2 virtual base %plural{1:class|:classes}2|"`。
- **L661 EN**: Continues the surrounding expression or declaration: `"%ordinal2 base class with different type %3|"`.
  **L661 CN**: 继续构造周围的表达式或声明：`"%ordinal2 base class with different type %3|"`。
- **L662 EN**: Continues the surrounding expression or declaration: `"%ordinal2 %select{non-virtual|virtual}3 base class %4|"`.
  **L662 CN**: 继续构造周围的表达式或声明：`"%ordinal2 %select{non-virtual|virtual}3 base class %4|"`。
- **L663 EN**: Continues the surrounding expression or declaration: `"%ordinal2 base class %3 with "`.
  **L663 CN**: 继续构造周围的表达式或声明：`"%ordinal2 base class %3 with "`。
- **L664 EN**: Adds a standalone statement or declaration: `"%select{public|protected|private|no}4 access specifier}1">;`.
  **L664 CN**: 添加一条独立语句或声明：`"%select{public|protected|private|no}4 access specifier}1">;`。
- **L665 EN**: Blank line separating nearby declarations or logic blocks.
  **L665 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L666 EN**: Declares TableGen def record `err_module_odr_violation_objc_interface`.
  **L666 CN**: 声明 TableGen def 记录 `err_module_odr_violation_objc_interface`。
- **L667 EN**: Continues the surrounding expression or declaration: `"%0 has different definitions in different modules; first difference is "`.
  **L667 CN**: 继续构造周围的表达式或声明：`"%0 has different definitions in different modules; first difference is "`。
- **L668 EN**: Continues the surrounding expression or declaration: `"%select{definition in module '%2'|defined here}1 found "`.
  **L668 CN**: 继续构造周围的表达式或声明：`"%select{definition in module '%2'|defined here}1 found "`。
- **L669 EN**: Continues the surrounding expression or declaration: `"%select{"`.
  **L669 CN**: 继续构造周围的表达式或声明：`"%select{"`。
- **L670 EN**: Continues the surrounding expression or declaration: `"%select{no super class|super class with type %5}4|"`.
  **L670 CN**: 继续构造周围的表达式或声明：`"%select{no super class|super class with type %5}4|"`。
- **L671 EN**: Continues the surrounding expression or declaration: `"instance variable '%4' access control is "`.
  **L671 CN**: 继续构造周围的表达式或声明：`"instance variable '%4' access control is "`。
- **L672 EN**: Continues the surrounding expression or declaration: `"%select{|@private|@protected|@public|@package}5"`.
  **L672 CN**: 继续构造周围的表达式或声明：`"%select{|@private|@protected|@public|@package}5"`。

### Lines 673-696

````tablegen
  "}3">;
def note_module_odr_violation_objc_interface : Note <
  "but in %select{'%1'|definition here}0 found "
  "%select{"
  "%select{no super class|super class with type %4}3|"
  "instance variable '%3' access control is "
    "%select{|@private|@protected|@public|@package}4"
  "}2">;

def err_module_odr_violation_template_parameter : Error <
  "%q0 has different definitions in different modules; first difference is "
  "%select{definition in module '%2'|defined here}1 found "
  "%select{"
  "unnamed template parameter|"
  "template parameter %5|"
  "template parameter with %select{no |}4default argument|"
  "template parameter with default argument}3">;

def note_module_odr_violation_template_parameter : Note <
  "but in '%0' found "
  "%select{"
  "unnamed template parameter %2|"
  "template parameter %3|"
  "template parameter with %select{no |}2default argument|"
````
- **L673 EN**: Adds a standalone statement or declaration: `"}3">;`.
  **L673 CN**: 添加一条独立语句或声明：`"}3">;`。
- **L674 EN**: Declares TableGen def record `note_module_odr_violation_objc_interface`.
  **L674 CN**: 声明 TableGen def 记录 `note_module_odr_violation_objc_interface`。
- **L675 EN**: Continues the surrounding expression or declaration: `"but in %select{'%1'|definition here}0 found "`.
  **L675 CN**: 继续构造周围的表达式或声明：`"but in %select{'%1'|definition here}0 found "`。
- **L676 EN**: Continues the surrounding expression or declaration: `"%select{"`.
  **L676 CN**: 继续构造周围的表达式或声明：`"%select{"`。
- **L677 EN**: Continues the surrounding expression or declaration: `"%select{no super class|super class with type %4}3|"`.
  **L677 CN**: 继续构造周围的表达式或声明：`"%select{no super class|super class with type %4}3|"`。
- **L678 EN**: Continues the surrounding expression or declaration: `"instance variable '%3' access control is "`.
  **L678 CN**: 继续构造周围的表达式或声明：`"instance variable '%3' access control is "`。
- **L679 EN**: Continues the surrounding expression or declaration: `"%select{|@private|@protected|@public|@package}4"`.
  **L679 CN**: 继续构造周围的表达式或声明：`"%select{|@private|@protected|@public|@package}4"`。
- **L680 EN**: Adds a standalone statement or declaration: `"}2">;`.
  **L680 CN**: 添加一条独立语句或声明：`"}2">;`。
- **L681 EN**: Blank line separating nearby declarations or logic blocks.
  **L681 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L682 EN**: Declares TableGen def record `err_module_odr_violation_template_parameter`.
  **L682 CN**: 声明 TableGen def 记录 `err_module_odr_violation_template_parameter`。
- **L683 EN**: Continues the surrounding expression or declaration: `"%q0 has different definitions in different modules; first difference is "`.
  **L683 CN**: 继续构造周围的表达式或声明：`"%q0 has different definitions in different modules; first difference is "`。
- **L684 EN**: Continues the surrounding expression or declaration: `"%select{definition in module '%2'|defined here}1 found "`.
  **L684 CN**: 继续构造周围的表达式或声明：`"%select{definition in module '%2'|defined here}1 found "`。
- **L685 EN**: Continues the surrounding expression or declaration: `"%select{"`.
  **L685 CN**: 继续构造周围的表达式或声明：`"%select{"`。
- **L686 EN**: Continues the surrounding expression or declaration: `"unnamed template parameter|"`.
  **L686 CN**: 继续构造周围的表达式或声明：`"unnamed template parameter|"`。
- **L687 EN**: Continues the surrounding expression or declaration: `"template parameter %5|"`.
  **L687 CN**: 继续构造周围的表达式或声明：`"template parameter %5|"`。
- **L688 EN**: Continues the surrounding expression or declaration: `"template parameter with %select{no |}4default argument|"`.
  **L688 CN**: 继续构造周围的表达式或声明：`"template parameter with %select{no |}4default argument|"`。
- **L689 EN**: Adds a standalone statement or declaration: `"template parameter with default argument}3">;`.
  **L689 CN**: 添加一条独立语句或声明：`"template parameter with default argument}3">;`。
- **L690 EN**: Blank line separating nearby declarations or logic blocks.
  **L690 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L691 EN**: Declares TableGen def record `note_module_odr_violation_template_parameter`.
  **L691 CN**: 声明 TableGen def 记录 `note_module_odr_violation_template_parameter`。
- **L692 EN**: Continues the surrounding expression or declaration: `"but in '%0' found "`.
  **L692 CN**: 继续构造周围的表达式或声明：`"but in '%0' found "`。
- **L693 EN**: Continues the surrounding expression or declaration: `"%select{"`.
  **L693 CN**: 继续构造周围的表达式或声明：`"%select{"`。
- **L694 EN**: Continues the surrounding expression or declaration: `"unnamed template parameter %2|"`.
  **L694 CN**: 继续构造周围的表达式或声明：`"unnamed template parameter %2|"`。
- **L695 EN**: Continues the surrounding expression or declaration: `"template parameter %3|"`.
  **L695 CN**: 继续构造周围的表达式或声明：`"template parameter %3|"`。
- **L696 EN**: Continues the surrounding expression or declaration: `"template parameter with %select{no |}2default argument|"`.
  **L696 CN**: 继续构造周围的表达式或声明：`"template parameter with %select{no |}2default argument|"`。

### Lines 697-720

````tablegen
  "template parameter with different default argument}1">;

def err_module_odr_violation_mismatch_decl : Error<
  "%q0 has different definitions in different modules; first difference is "
  "%select{definition in module '%2'|defined here}1 found "
  "%select{end of class|public access specifier|private access specifier|"
  "protected access specifier|static assert|field|method|type alias|typedef|"
  "data member|friend declaration|function template|method|instance variable|"
  "property}3">;
def note_module_odr_violation_mismatch_decl : Note<
  "but in %select{'%1'|definition here}0 found "
  "%select{end of class|public access specifier|private access specifier|"
  "protected access specifier|static assert|field|method|type alias|typedef|"
  "data member|friend declaration|function template|method|instance variable|"
  "property}2">;

def err_module_odr_violation_record : Error<
  "%q0 has different definitions in different modules; first difference is "
  "%select{definition in module '%2'|defined here}1 found "
  "%select{"
  "static assert with condition|"
  "static assert with message|"
  "static assert with %select{|no }4message|"
  "%select{method %5|constructor|destructor}4|"
````
- **L697 EN**: Adds a standalone statement or declaration: `"template parameter with different default argument}1">;`.
  **L697 CN**: 添加一条独立语句或声明：`"template parameter with different default argument}1">;`。
- **L698 EN**: Blank line separating nearby declarations or logic blocks.
  **L698 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L699 EN**: Declares TableGen def record `err_module_odr_violation_mismatch_decl`.
  **L699 CN**: 声明 TableGen def 记录 `err_module_odr_violation_mismatch_decl`。
- **L700 EN**: Continues the surrounding expression or declaration: `"%q0 has different definitions in different modules; first difference is "`.
  **L700 CN**: 继续构造周围的表达式或声明：`"%q0 has different definitions in different modules; first difference is "`。
- **L701 EN**: Continues the surrounding expression or declaration: `"%select{definition in module '%2'|defined here}1 found "`.
  **L701 CN**: 继续构造周围的表达式或声明：`"%select{definition in module '%2'|defined here}1 found "`。
- **L702 EN**: Continues the surrounding expression or declaration: `"%select{end of class|public access specifier|private access specifier|"`.
  **L702 CN**: 继续构造周围的表达式或声明：`"%select{end of class|public access specifier|private access specifier|"`。
- **L703 EN**: Continues the surrounding expression or declaration: `"protected access specifier|static assert|field|method|type alias|typedef|"`.
  **L703 CN**: 继续构造周围的表达式或声明：`"protected access specifier|static assert|field|method|type alias|typedef|"`。
- **L704 EN**: Continues the surrounding expression or declaration: `"data member|friend declaration|function template|method|instance variable|"`.
  **L704 CN**: 继续构造周围的表达式或声明：`"data member|friend declaration|function template|method|instance variable|"`。
- **L705 EN**: Adds a standalone statement or declaration: `"property}3">;`.
  **L705 CN**: 添加一条独立语句或声明：`"property}3">;`。
- **L706 EN**: Declares TableGen def record `note_module_odr_violation_mismatch_decl`.
  **L706 CN**: 声明 TableGen def 记录 `note_module_odr_violation_mismatch_decl`。
- **L707 EN**: Continues the surrounding expression or declaration: `"but in %select{'%1'|definition here}0 found "`.
  **L707 CN**: 继续构造周围的表达式或声明：`"but in %select{'%1'|definition here}0 found "`。
- **L708 EN**: Continues the surrounding expression or declaration: `"%select{end of class|public access specifier|private access specifier|"`.
  **L708 CN**: 继续构造周围的表达式或声明：`"%select{end of class|public access specifier|private access specifier|"`。
- **L709 EN**: Continues the surrounding expression or declaration: `"protected access specifier|static assert|field|method|type alias|typedef|"`.
  **L709 CN**: 继续构造周围的表达式或声明：`"protected access specifier|static assert|field|method|type alias|typedef|"`。
- **L710 EN**: Continues the surrounding expression or declaration: `"data member|friend declaration|function template|method|instance variable|"`.
  **L710 CN**: 继续构造周围的表达式或声明：`"data member|friend declaration|function template|method|instance variable|"`。
- **L711 EN**: Adds a standalone statement or declaration: `"property}2">;`.
  **L711 CN**: 添加一条独立语句或声明：`"property}2">;`。
- **L712 EN**: Blank line separating nearby declarations or logic blocks.
  **L712 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L713 EN**: Declares TableGen def record `err_module_odr_violation_record`.
  **L713 CN**: 声明 TableGen def 记录 `err_module_odr_violation_record`。
- **L714 EN**: Continues the surrounding expression or declaration: `"%q0 has different definitions in different modules; first difference is "`.
  **L714 CN**: 继续构造周围的表达式或声明：`"%q0 has different definitions in different modules; first difference is "`。
- **L715 EN**: Continues the surrounding expression or declaration: `"%select{definition in module '%2'|defined here}1 found "`.
  **L715 CN**: 继续构造周围的表达式或声明：`"%select{definition in module '%2'|defined here}1 found "`。
- **L716 EN**: Continues the surrounding expression or declaration: `"%select{"`.
  **L716 CN**: 继续构造周围的表达式或声明：`"%select{"`。
- **L717 EN**: Continues the surrounding expression or declaration: `"static assert with condition|"`.
  **L717 CN**: 继续构造周围的表达式或声明：`"static assert with condition|"`。
- **L718 EN**: Continues the surrounding expression or declaration: `"static assert with message|"`.
  **L718 CN**: 继续构造周围的表达式或声明：`"static assert with message|"`。
- **L719 EN**: Continues the surrounding expression or declaration: `"static assert with %select{|no }4message|"`.
  **L719 CN**: 继续构造周围的表达式或声明：`"static assert with %select{|no }4message|"`。
- **L720 EN**: Continues the surrounding expression or declaration: `"%select{method %5|constructor|destructor}4|"`.
  **L720 CN**: 继续构造周围的表达式或声明：`"%select{method %5|constructor|destructor}4|"`。

### Lines 721-744

````tablegen
  "%select{method %5|constructor|destructor}4 "
    "is %select{not deleted|deleted}6|"
  "%select{method %5|constructor|destructor}4 "
    "is %select{not defaulted|defaulted}6|"
  "%select{method %5|constructor|destructor}4 "
    "is %select{|pure }6%select{not virtual|virtual}7|"
  "%select{method %5|constructor|destructor}4 "
    "is %select{not static|static}6|"
  "%select{method %5|constructor|destructor}4 "
    "is %select{not volatile|volatile}6|"
  "%select{method %5|constructor|destructor}4 "
    "is %select{not const|const}6|"
  "%select{method %5|constructor|destructor}4 "
    "is %select{not inline|inline}6|"
  "%select{method %5|constructor|destructor}4 "
    "with %ordinal6 parameter with%select{out|}7 a default argument|"
  "%select{method %5|constructor|destructor}4 "
    "with %ordinal6 parameter with a default argument|"
  "%select{method %5|constructor|destructor}4 "
    "with %select{no |}6template arguments|"
  "%select{method %5|constructor|destructor}4 "
    "with %6 template argument%s6|"
  "%select{method %5|constructor|destructor}4 "
    "with %6 for %ordinal7 template argument|"
````
- **L721 EN**: Continues the surrounding expression or declaration: `"%select{method %5|constructor|destructor}4 "`.
  **L721 CN**: 继续构造周围的表达式或声明：`"%select{method %5|constructor|destructor}4 "`。
- **L722 EN**: Continues the surrounding expression or declaration: `"is %select{not deleted|deleted}6|"`.
  **L722 CN**: 继续构造周围的表达式或声明：`"is %select{not deleted|deleted}6|"`。
- **L723 EN**: Continues the surrounding expression or declaration: `"%select{method %5|constructor|destructor}4 "`.
  **L723 CN**: 继续构造周围的表达式或声明：`"%select{method %5|constructor|destructor}4 "`。
- **L724 EN**: Continues the surrounding expression or declaration: `"is %select{not defaulted|defaulted}6|"`.
  **L724 CN**: 继续构造周围的表达式或声明：`"is %select{not defaulted|defaulted}6|"`。
- **L725 EN**: Continues the surrounding expression or declaration: `"%select{method %5|constructor|destructor}4 "`.
  **L725 CN**: 继续构造周围的表达式或声明：`"%select{method %5|constructor|destructor}4 "`。
- **L726 EN**: Continues the surrounding expression or declaration: `"is %select{|pure }6%select{not virtual|virtual}7|"`.
  **L726 CN**: 继续构造周围的表达式或声明：`"is %select{|pure }6%select{not virtual|virtual}7|"`。
- **L727 EN**: Continues the surrounding expression or declaration: `"%select{method %5|constructor|destructor}4 "`.
  **L727 CN**: 继续构造周围的表达式或声明：`"%select{method %5|constructor|destructor}4 "`。
- **L728 EN**: Continues the surrounding expression or declaration: `"is %select{not static|static}6|"`.
  **L728 CN**: 继续构造周围的表达式或声明：`"is %select{not static|static}6|"`。
- **L729 EN**: Continues the surrounding expression or declaration: `"%select{method %5|constructor|destructor}4 "`.
  **L729 CN**: 继续构造周围的表达式或声明：`"%select{method %5|constructor|destructor}4 "`。
- **L730 EN**: Continues the surrounding expression or declaration: `"is %select{not volatile|volatile}6|"`.
  **L730 CN**: 继续构造周围的表达式或声明：`"is %select{not volatile|volatile}6|"`。
- **L731 EN**: Continues the surrounding expression or declaration: `"%select{method %5|constructor|destructor}4 "`.
  **L731 CN**: 继续构造周围的表达式或声明：`"%select{method %5|constructor|destructor}4 "`。
- **L732 EN**: Continues the surrounding expression or declaration: `"is %select{not const|const}6|"`.
  **L732 CN**: 继续构造周围的表达式或声明：`"is %select{not const|const}6|"`。
- **L733 EN**: Continues the surrounding expression or declaration: `"%select{method %5|constructor|destructor}4 "`.
  **L733 CN**: 继续构造周围的表达式或声明：`"%select{method %5|constructor|destructor}4 "`。
- **L734 EN**: Continues the surrounding expression or declaration: `"is %select{not inline|inline}6|"`.
  **L734 CN**: 继续构造周围的表达式或声明：`"is %select{not inline|inline}6|"`。
- **L735 EN**: Continues the surrounding expression or declaration: `"%select{method %5|constructor|destructor}4 "`.
  **L735 CN**: 继续构造周围的表达式或声明：`"%select{method %5|constructor|destructor}4 "`。
- **L736 EN**: Continues the surrounding expression or declaration: `"with %ordinal6 parameter with%select{out|}7 a default argument|"`.
  **L736 CN**: 继续构造周围的表达式或声明：`"with %ordinal6 parameter with%select{out|}7 a default argument|"`。
- **L737 EN**: Continues the surrounding expression or declaration: `"%select{method %5|constructor|destructor}4 "`.
  **L737 CN**: 继续构造周围的表达式或声明：`"%select{method %5|constructor|destructor}4 "`。
- **L738 EN**: Continues the surrounding expression or declaration: `"with %ordinal6 parameter with a default argument|"`.
  **L738 CN**: 继续构造周围的表达式或声明：`"with %ordinal6 parameter with a default argument|"`。
- **L739 EN**: Continues the surrounding expression or declaration: `"%select{method %5|constructor|destructor}4 "`.
  **L739 CN**: 继续构造周围的表达式或声明：`"%select{method %5|constructor|destructor}4 "`。
- **L740 EN**: Continues the surrounding expression or declaration: `"with %select{no |}6template arguments|"`.
  **L740 CN**: 继续构造周围的表达式或声明：`"with %select{no |}6template arguments|"`。
- **L741 EN**: Continues the surrounding expression or declaration: `"%select{method %5|constructor|destructor}4 "`.
  **L741 CN**: 继续构造周围的表达式或声明：`"%select{method %5|constructor|destructor}4 "`。
- **L742 EN**: Continues the surrounding expression or declaration: `"with %6 template argument%s6|"`.
  **L742 CN**: 继续构造周围的表达式或声明：`"with %6 template argument%s6|"`。
- **L743 EN**: Continues the surrounding expression or declaration: `"%select{method %5|constructor|destructor}4 "`.
  **L743 CN**: 继续构造周围的表达式或声明：`"%select{method %5|constructor|destructor}4 "`。
- **L744 EN**: Continues the surrounding expression or declaration: `"with %6 for %ordinal7 template argument|"`.
  **L744 CN**: 继续构造周围的表达式或声明：`"with %6 for %ordinal7 template argument|"`。

### Lines 745-768

````tablegen
  "%select{method %5|constructor|destructor}4 "
    "with %select{no body|body}6|"
  "%select{method %5|constructor|destructor}4 "
    "with body|"
  "friend %select{class|function}4|"
  "friend %4|"
  "friend function %4|"
  "function template %4 with %5 template parameter%s5|"
  "function template %4 with %ordinal5 template parameter being a "
    "%select{type|non-type|template}6 template parameter|"
  "function template %4 with %ordinal5 template parameter "
    "%select{with no name|named %7}6|"
  "function template %4 with %ordinal5 template parameter with "
    "%select{no |}6default argument|"
  "function template %4 with %ordinal5 template parameter with "
    "default argument %6|"
  "function template %4 with %ordinal5 template parameter with one type|"
  "function template %4 with %ordinal5 template parameter %select{not |}6"
    "being a template parameter pack|"
  "}3">;

def note_module_odr_violation_record : Note<"but in '%0' found "
  "%select{"
  "static assert with different condition|"
````
- **L745 EN**: Continues the surrounding expression or declaration: `"%select{method %5|constructor|destructor}4 "`.
  **L745 CN**: 继续构造周围的表达式或声明：`"%select{method %5|constructor|destructor}4 "`。
- **L746 EN**: Continues the surrounding expression or declaration: `"with %select{no body|body}6|"`.
  **L746 CN**: 继续构造周围的表达式或声明：`"with %select{no body|body}6|"`。
- **L747 EN**: Continues the surrounding expression or declaration: `"%select{method %5|constructor|destructor}4 "`.
  **L747 CN**: 继续构造周围的表达式或声明：`"%select{method %5|constructor|destructor}4 "`。
- **L748 EN**: Continues the surrounding expression or declaration: `"with body|"`.
  **L748 CN**: 继续构造周围的表达式或声明：`"with body|"`。
- **L749 EN**: Continues the surrounding expression or declaration: `"friend %select{class|function}4|"`.
  **L749 CN**: 继续构造周围的表达式或声明：`"friend %select{class|function}4|"`。
- **L750 EN**: Continues the surrounding expression or declaration: `"friend %4|"`.
  **L750 CN**: 继续构造周围的表达式或声明：`"friend %4|"`。
- **L751 EN**: Continues the surrounding expression or declaration: `"friend function %4|"`.
  **L751 CN**: 继续构造周围的表达式或声明：`"friend function %4|"`。
- **L752 EN**: Continues the surrounding expression or declaration: `"function template %4 with %5 template parameter%s5|"`.
  **L752 CN**: 继续构造周围的表达式或声明：`"function template %4 with %5 template parameter%s5|"`。
- **L753 EN**: Continues the surrounding expression or declaration: `"function template %4 with %ordinal5 template parameter being a "`.
  **L753 CN**: 继续构造周围的表达式或声明：`"function template %4 with %ordinal5 template parameter being a "`。
- **L754 EN**: Continues the surrounding expression or declaration: `"%select{type|non-type|template}6 template parameter|"`.
  **L754 CN**: 继续构造周围的表达式或声明：`"%select{type|non-type|template}6 template parameter|"`。
- **L755 EN**: Continues the surrounding expression or declaration: `"function template %4 with %ordinal5 template parameter "`.
  **L755 CN**: 继续构造周围的表达式或声明：`"function template %4 with %ordinal5 template parameter "`。
- **L756 EN**: Continues the surrounding expression or declaration: `"%select{with no name|named %7}6|"`.
  **L756 CN**: 继续构造周围的表达式或声明：`"%select{with no name|named %7}6|"`。
- **L757 EN**: Continues the surrounding expression or declaration: `"function template %4 with %ordinal5 template parameter with "`.
  **L757 CN**: 继续构造周围的表达式或声明：`"function template %4 with %ordinal5 template parameter with "`。
- **L758 EN**: Continues the surrounding expression or declaration: `"%select{no |}6default argument|"`.
  **L758 CN**: 继续构造周围的表达式或声明：`"%select{no |}6default argument|"`。
- **L759 EN**: Continues the surrounding expression or declaration: `"function template %4 with %ordinal5 template parameter with "`.
  **L759 CN**: 继续构造周围的表达式或声明：`"function template %4 with %ordinal5 template parameter with "`。
- **L760 EN**: Continues the surrounding expression or declaration: `"default argument %6|"`.
  **L760 CN**: 继续构造周围的表达式或声明：`"default argument %6|"`。
- **L761 EN**: Continues the surrounding expression or declaration: `"function template %4 with %ordinal5 template parameter with one type|"`.
  **L761 CN**: 继续构造周围的表达式或声明：`"function template %4 with %ordinal5 template parameter with one type|"`。
- **L762 EN**: Continues the surrounding expression or declaration: `"function template %4 with %ordinal5 template parameter %select{not |}6"`.
  **L762 CN**: 继续构造周围的表达式或声明：`"function template %4 with %ordinal5 template parameter %select{not |}6"`。
- **L763 EN**: Continues the surrounding expression or declaration: `"being a template parameter pack|"`.
  **L763 CN**: 继续构造周围的表达式或声明：`"being a template parameter pack|"`。
- **L764 EN**: Adds a standalone statement or declaration: `"}3">;`.
  **L764 CN**: 添加一条独立语句或声明：`"}3">;`。
- **L765 EN**: Blank line separating nearby declarations or logic blocks.
  **L765 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L766 EN**: Declares TableGen def record `note_module_odr_violation_record`.
  **L766 CN**: 声明 TableGen def 记录 `note_module_odr_violation_record`。
- **L767 EN**: Continues the surrounding expression or declaration: `"%select{"`.
  **L767 CN**: 继续构造周围的表达式或声明：`"%select{"`。
- **L768 EN**: Continues the surrounding expression or declaration: `"static assert with different condition|"`.
  **L768 CN**: 继续构造周围的表达式或声明：`"static assert with different condition|"`。

### Lines 769-792

````tablegen
  "static assert with different message|"
  "static assert with %select{|no }2message|"
  "%select{method %3|constructor|destructor}2|"
  "%select{method %3|constructor|destructor}2 "
    "is %select{not deleted|deleted}4|"
  "%select{method %3|constructor|destructor}2 "
    "is %select{not defaulted|defaulted}4|"
  "%select{method %3|constructor|destructor}2 "
    "is %select{|pure }4%select{not virtual|virtual}5|"
  "%select{method %3|constructor|destructor}2 "
    "is %select{not static|static}4|"
  "%select{method %3|constructor|destructor}2 "
    "is %select{not volatile|volatile}4|"
  "%select{method %3|constructor|destructor}2 "
    "is %select{not const|const}4|"
  "%select{method %3|constructor|destructor}2 "
    "is %select{not inline|inline}4|"
  "%select{method %3|constructor|destructor}2 "
    "with %ordinal4 parameter with%select{out|}5 a default argument|"
  "%select{method %3|constructor|destructor}2 "
    "with %ordinal4 parameter with a different default argument|"
  "%select{method %3|constructor|destructor}2 "
    "with %select{no |}4template arguments|"
  "%select{method %3|constructor|destructor}2 "
````
- **L769 EN**: Continues the surrounding expression or declaration: `"static assert with different message|"`.
  **L769 CN**: 继续构造周围的表达式或声明：`"static assert with different message|"`。
- **L770 EN**: Continues the surrounding expression or declaration: `"static assert with %select{|no }2message|"`.
  **L770 CN**: 继续构造周围的表达式或声明：`"static assert with %select{|no }2message|"`。
- **L771 EN**: Continues the surrounding expression or declaration: `"%select{method %3|constructor|destructor}2|"`.
  **L771 CN**: 继续构造周围的表达式或声明：`"%select{method %3|constructor|destructor}2|"`。
- **L772 EN**: Continues the surrounding expression or declaration: `"%select{method %3|constructor|destructor}2 "`.
  **L772 CN**: 继续构造周围的表达式或声明：`"%select{method %3|constructor|destructor}2 "`。
- **L773 EN**: Continues the surrounding expression or declaration: `"is %select{not deleted|deleted}4|"`.
  **L773 CN**: 继续构造周围的表达式或声明：`"is %select{not deleted|deleted}4|"`。
- **L774 EN**: Continues the surrounding expression or declaration: `"%select{method %3|constructor|destructor}2 "`.
  **L774 CN**: 继续构造周围的表达式或声明：`"%select{method %3|constructor|destructor}2 "`。
- **L775 EN**: Continues the surrounding expression or declaration: `"is %select{not defaulted|defaulted}4|"`.
  **L775 CN**: 继续构造周围的表达式或声明：`"is %select{not defaulted|defaulted}4|"`。
- **L776 EN**: Continues the surrounding expression or declaration: `"%select{method %3|constructor|destructor}2 "`.
  **L776 CN**: 继续构造周围的表达式或声明：`"%select{method %3|constructor|destructor}2 "`。
- **L777 EN**: Continues the surrounding expression or declaration: `"is %select{|pure }4%select{not virtual|virtual}5|"`.
  **L777 CN**: 继续构造周围的表达式或声明：`"is %select{|pure }4%select{not virtual|virtual}5|"`。
- **L778 EN**: Continues the surrounding expression or declaration: `"%select{method %3|constructor|destructor}2 "`.
  **L778 CN**: 继续构造周围的表达式或声明：`"%select{method %3|constructor|destructor}2 "`。
- **L779 EN**: Continues the surrounding expression or declaration: `"is %select{not static|static}4|"`.
  **L779 CN**: 继续构造周围的表达式或声明：`"is %select{not static|static}4|"`。
- **L780 EN**: Continues the surrounding expression or declaration: `"%select{method %3|constructor|destructor}2 "`.
  **L780 CN**: 继续构造周围的表达式或声明：`"%select{method %3|constructor|destructor}2 "`。
- **L781 EN**: Continues the surrounding expression or declaration: `"is %select{not volatile|volatile}4|"`.
  **L781 CN**: 继续构造周围的表达式或声明：`"is %select{not volatile|volatile}4|"`。
- **L782 EN**: Continues the surrounding expression or declaration: `"%select{method %3|constructor|destructor}2 "`.
  **L782 CN**: 继续构造周围的表达式或声明：`"%select{method %3|constructor|destructor}2 "`。
- **L783 EN**: Continues the surrounding expression or declaration: `"is %select{not const|const}4|"`.
  **L783 CN**: 继续构造周围的表达式或声明：`"is %select{not const|const}4|"`。
- **L784 EN**: Continues the surrounding expression or declaration: `"%select{method %3|constructor|destructor}2 "`.
  **L784 CN**: 继续构造周围的表达式或声明：`"%select{method %3|constructor|destructor}2 "`。
- **L785 EN**: Continues the surrounding expression or declaration: `"is %select{not inline|inline}4|"`.
  **L785 CN**: 继续构造周围的表达式或声明：`"is %select{not inline|inline}4|"`。
- **L786 EN**: Continues the surrounding expression or declaration: `"%select{method %3|constructor|destructor}2 "`.
  **L786 CN**: 继续构造周围的表达式或声明：`"%select{method %3|constructor|destructor}2 "`。
- **L787 EN**: Continues the surrounding expression or declaration: `"with %ordinal4 parameter with%select{out|}5 a default argument|"`.
  **L787 CN**: 继续构造周围的表达式或声明：`"with %ordinal4 parameter with%select{out|}5 a default argument|"`。
- **L788 EN**: Continues the surrounding expression or declaration: `"%select{method %3|constructor|destructor}2 "`.
  **L788 CN**: 继续构造周围的表达式或声明：`"%select{method %3|constructor|destructor}2 "`。
- **L789 EN**: Continues the surrounding expression or declaration: `"with %ordinal4 parameter with a different default argument|"`.
  **L789 CN**: 继续构造周围的表达式或声明：`"with %ordinal4 parameter with a different default argument|"`。
- **L790 EN**: Continues the surrounding expression or declaration: `"%select{method %3|constructor|destructor}2 "`.
  **L790 CN**: 继续构造周围的表达式或声明：`"%select{method %3|constructor|destructor}2 "`。
- **L791 EN**: Continues the surrounding expression or declaration: `"with %select{no |}4template arguments|"`.
  **L791 CN**: 继续构造周围的表达式或声明：`"with %select{no |}4template arguments|"`。
- **L792 EN**: Continues the surrounding expression or declaration: `"%select{method %3|constructor|destructor}2 "`.
  **L792 CN**: 继续构造周围的表达式或声明：`"%select{method %3|constructor|destructor}2 "`。

### Lines 793-816

````tablegen
    "with %4 template argument%s4|"
  "%select{method %3|constructor|destructor}2 "
    "with %4 for %ordinal5 template argument|"
  "%select{method %3|constructor|destructor}2 "
    "with %select{no body|body}4|"
  "%select{method %3|constructor|destructor}2 "
    "with different body|"
  "friend %select{class|function}2|"
  "friend %2|"
  "friend function %2|"
  "function template %2 with %3 template parameter%s3|"
  "function template %2 with %ordinal3 template paramter being a "
    "%select{type|non-type|template}4 template parameter|"
  "function template %2 with %ordinal3 template parameter "
    "%select{with no name|named %5}4|"
  "function template %2 with %ordinal3 template parameter with "
    "%select{no |}4default argument|"
  "function template %2 with %ordinal3 template parameter with "
    "default argument %4|"
  "function template %2 with %ordinal3 template parameter with different type|"
  "function template %2 with %ordinal3 template parameter %select{not |}4"
    "being a template parameter pack|"
  "}1">;

````
- **L793 EN**: Continues the surrounding expression or declaration: `"with %4 template argument%s4|"`.
  **L793 CN**: 继续构造周围的表达式或声明：`"with %4 template argument%s4|"`。
- **L794 EN**: Continues the surrounding expression or declaration: `"%select{method %3|constructor|destructor}2 "`.
  **L794 CN**: 继续构造周围的表达式或声明：`"%select{method %3|constructor|destructor}2 "`。
- **L795 EN**: Continues the surrounding expression or declaration: `"with %4 for %ordinal5 template argument|"`.
  **L795 CN**: 继续构造周围的表达式或声明：`"with %4 for %ordinal5 template argument|"`。
- **L796 EN**: Continues the surrounding expression or declaration: `"%select{method %3|constructor|destructor}2 "`.
  **L796 CN**: 继续构造周围的表达式或声明：`"%select{method %3|constructor|destructor}2 "`。
- **L797 EN**: Continues the surrounding expression or declaration: `"with %select{no body|body}4|"`.
  **L797 CN**: 继续构造周围的表达式或声明：`"with %select{no body|body}4|"`。
- **L798 EN**: Continues the surrounding expression or declaration: `"%select{method %3|constructor|destructor}2 "`.
  **L798 CN**: 继续构造周围的表达式或声明：`"%select{method %3|constructor|destructor}2 "`。
- **L799 EN**: Continues the surrounding expression or declaration: `"with different body|"`.
  **L799 CN**: 继续构造周围的表达式或声明：`"with different body|"`。
- **L800 EN**: Continues the surrounding expression or declaration: `"friend %select{class|function}2|"`.
  **L800 CN**: 继续构造周围的表达式或声明：`"friend %select{class|function}2|"`。
- **L801 EN**: Continues the surrounding expression or declaration: `"friend %2|"`.
  **L801 CN**: 继续构造周围的表达式或声明：`"friend %2|"`。
- **L802 EN**: Continues the surrounding expression or declaration: `"friend function %2|"`.
  **L802 CN**: 继续构造周围的表达式或声明：`"friend function %2|"`。
- **L803 EN**: Continues the surrounding expression or declaration: `"function template %2 with %3 template parameter%s3|"`.
  **L803 CN**: 继续构造周围的表达式或声明：`"function template %2 with %3 template parameter%s3|"`。
- **L804 EN**: Continues the surrounding expression or declaration: `"function template %2 with %ordinal3 template paramter being a "`.
  **L804 CN**: 继续构造周围的表达式或声明：`"function template %2 with %ordinal3 template paramter being a "`。
- **L805 EN**: Continues the surrounding expression or declaration: `"%select{type|non-type|template}4 template parameter|"`.
  **L805 CN**: 继续构造周围的表达式或声明：`"%select{type|non-type|template}4 template parameter|"`。
- **L806 EN**: Continues the surrounding expression or declaration: `"function template %2 with %ordinal3 template parameter "`.
  **L806 CN**: 继续构造周围的表达式或声明：`"function template %2 with %ordinal3 template parameter "`。
- **L807 EN**: Continues the surrounding expression or declaration: `"%select{with no name|named %5}4|"`.
  **L807 CN**: 继续构造周围的表达式或声明：`"%select{with no name|named %5}4|"`。
- **L808 EN**: Continues the surrounding expression or declaration: `"function template %2 with %ordinal3 template parameter with "`.
  **L808 CN**: 继续构造周围的表达式或声明：`"function template %2 with %ordinal3 template parameter with "`。
- **L809 EN**: Continues the surrounding expression or declaration: `"%select{no |}4default argument|"`.
  **L809 CN**: 继续构造周围的表达式或声明：`"%select{no |}4default argument|"`。
- **L810 EN**: Continues the surrounding expression or declaration: `"function template %2 with %ordinal3 template parameter with "`.
  **L810 CN**: 继续构造周围的表达式或声明：`"function template %2 with %ordinal3 template parameter with "`。
- **L811 EN**: Continues the surrounding expression or declaration: `"default argument %4|"`.
  **L811 CN**: 继续构造周围的表达式或声明：`"default argument %4|"`。
- **L812 EN**: Continues the surrounding expression or declaration: `"function template %2 with %ordinal3 template parameter with different type|"`.
  **L812 CN**: 继续构造周围的表达式或声明：`"function template %2 with %ordinal3 template parameter with different type|"`。
- **L813 EN**: Continues the surrounding expression or declaration: `"function template %2 with %ordinal3 template parameter %select{not |}4"`.
  **L813 CN**: 继续构造周围的表达式或声明：`"function template %2 with %ordinal3 template parameter %select{not |}4"`。
- **L814 EN**: Continues the surrounding expression or declaration: `"being a template parameter pack|"`.
  **L814 CN**: 继续构造周围的表达式或声明：`"being a template parameter pack|"`。
- **L815 EN**: Adds a standalone statement or declaration: `"}1">;`.
  **L815 CN**: 添加一条独立语句或声明：`"}1">;`。
- **L816 EN**: Blank line separating nearby declarations or logic blocks.
  **L816 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 817-840

````tablegen
def err_module_odr_violation_field : Error<
  "%q0 has different definitions in different modules; first difference is "
  "%select{definition in module '%2'|defined here}1 found "
  "%select{"
  "field %4|"
  "field %4 with type %5|"
  "%select{non-|}5bit-field %4|"
  "bit-field %4 with one width expression|"
  "%select{non-|}5mutable field %4|"
  "field %4 with %select{no|an}5 initializer|"
  "field %4 with an initializer"
  "}3">;
def note_module_odr_violation_field : Note<
  "but in %select{'%1'|definition here}0 found "
  "%select{"
  "field %3|"
  "field %3 with type %4|"
  "%select{non-|}4bit-field %3|"
  "bit-field %3 with different width expression|"
  "%select{non-|}4mutable field %3|"
  "field %3 with %select{no|an}4 initializer|"
  "field %3 with a different initializer"
  "}2">;

````
- **L817 EN**: Declares TableGen def record `err_module_odr_violation_field`.
  **L817 CN**: 声明 TableGen def 记录 `err_module_odr_violation_field`。
- **L818 EN**: Continues the surrounding expression or declaration: `"%q0 has different definitions in different modules; first difference is "`.
  **L818 CN**: 继续构造周围的表达式或声明：`"%q0 has different definitions in different modules; first difference is "`。
- **L819 EN**: Continues the surrounding expression or declaration: `"%select{definition in module '%2'|defined here}1 found "`.
  **L819 CN**: 继续构造周围的表达式或声明：`"%select{definition in module '%2'|defined here}1 found "`。
- **L820 EN**: Continues the surrounding expression or declaration: `"%select{"`.
  **L820 CN**: 继续构造周围的表达式或声明：`"%select{"`。
- **L821 EN**: Continues the surrounding expression or declaration: `"field %4|"`.
  **L821 CN**: 继续构造周围的表达式或声明：`"field %4|"`。
- **L822 EN**: Continues the surrounding expression or declaration: `"field %4 with type %5|"`.
  **L822 CN**: 继续构造周围的表达式或声明：`"field %4 with type %5|"`。
- **L823 EN**: Continues the surrounding expression or declaration: `"%select{non-|}5bit-field %4|"`.
  **L823 CN**: 继续构造周围的表达式或声明：`"%select{non-|}5bit-field %4|"`。
- **L824 EN**: Continues the surrounding expression or declaration: `"bit-field %4 with one width expression|"`.
  **L824 CN**: 继续构造周围的表达式或声明：`"bit-field %4 with one width expression|"`。
- **L825 EN**: Continues the surrounding expression or declaration: `"%select{non-|}5mutable field %4|"`.
  **L825 CN**: 继续构造周围的表达式或声明：`"%select{non-|}5mutable field %4|"`。
- **L826 EN**: Continues the surrounding expression or declaration: `"field %4 with %select{no|an}5 initializer|"`.
  **L826 CN**: 继续构造周围的表达式或声明：`"field %4 with %select{no|an}5 initializer|"`。
- **L827 EN**: Continues the surrounding expression or declaration: `"field %4 with an initializer"`.
  **L827 CN**: 继续构造周围的表达式或声明：`"field %4 with an initializer"`。
- **L828 EN**: Adds a standalone statement or declaration: `"}3">;`.
  **L828 CN**: 添加一条独立语句或声明：`"}3">;`。
- **L829 EN**: Declares TableGen def record `note_module_odr_violation_field`.
  **L829 CN**: 声明 TableGen def 记录 `note_module_odr_violation_field`。
- **L830 EN**: Continues the surrounding expression or declaration: `"but in %select{'%1'|definition here}0 found "`.
  **L830 CN**: 继续构造周围的表达式或声明：`"but in %select{'%1'|definition here}0 found "`。
- **L831 EN**: Continues the surrounding expression or declaration: `"%select{"`.
  **L831 CN**: 继续构造周围的表达式或声明：`"%select{"`。
- **L832 EN**: Continues the surrounding expression or declaration: `"field %3|"`.
  **L832 CN**: 继续构造周围的表达式或声明：`"field %3|"`。
- **L833 EN**: Continues the surrounding expression or declaration: `"field %3 with type %4|"`.
  **L833 CN**: 继续构造周围的表达式或声明：`"field %3 with type %4|"`。
- **L834 EN**: Continues the surrounding expression or declaration: `"%select{non-|}4bit-field %3|"`.
  **L834 CN**: 继续构造周围的表达式或声明：`"%select{non-|}4bit-field %3|"`。
- **L835 EN**: Continues the surrounding expression or declaration: `"bit-field %3 with different width expression|"`.
  **L835 CN**: 继续构造周围的表达式或声明：`"bit-field %3 with different width expression|"`。
- **L836 EN**: Continues the surrounding expression or declaration: `"%select{non-|}4mutable field %3|"`.
  **L836 CN**: 继续构造周围的表达式或声明：`"%select{non-|}4mutable field %3|"`。
- **L837 EN**: Continues the surrounding expression or declaration: `"field %3 with %select{no|an}4 initializer|"`.
  **L837 CN**: 继续构造周围的表达式或声明：`"field %3 with %select{no|an}4 initializer|"`。
- **L838 EN**: Continues the surrounding expression or declaration: `"field %3 with a different initializer"`.
  **L838 CN**: 继续构造周围的表达式或声明：`"field %3 with a different initializer"`。
- **L839 EN**: Adds a standalone statement or declaration: `"}2">;`.
  **L839 CN**: 添加一条独立语句或声明：`"}2">;`。
- **L840 EN**: Blank line separating nearby declarations or logic blocks.
  **L840 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 841-864

````tablegen
def err_module_odr_violation_typedef : Error<
  "%q0 has different definitions in different modules; first difference is "
  "%select{definition in module '%2'|defined here}1 found "
  "%select{"
  "%select{typedef|type alias}4 name %5|"
  "%select{typedef|type alias}4 %5 with underlying type %6"
  "}3">;
def note_module_odr_violation_typedef : Note<"but in '%0' found "
  "%select{"
  "%select{typedef|type alias}2 name %3|"
  "%select{typedef|type alias}2 %3 with different underlying type %4"
  "}1">;

def err_module_odr_violation_variable : Error<
  "%q0 has different definitions in different modules; first difference is "
  "%select{definition in module '%2'|defined here}1 found "
  "%select{"
  "data member with name %4|"
  "data member %4 with type %5|"
  "data member %4 with%select{out|}5 an initializer|"
  "data member %4 with an initializer|"
  "data member %4 %select{is constexpr|is not constexpr}5"
  "}3">;
def note_module_odr_violation_variable : Note<"but in '%0' found "
````
- **L841 EN**: Declares TableGen def record `err_module_odr_violation_typedef`.
  **L841 CN**: 声明 TableGen def 记录 `err_module_odr_violation_typedef`。
- **L842 EN**: Continues the surrounding expression or declaration: `"%q0 has different definitions in different modules; first difference is "`.
  **L842 CN**: 继续构造周围的表达式或声明：`"%q0 has different definitions in different modules; first difference is "`。
- **L843 EN**: Continues the surrounding expression or declaration: `"%select{definition in module '%2'|defined here}1 found "`.
  **L843 CN**: 继续构造周围的表达式或声明：`"%select{definition in module '%2'|defined here}1 found "`。
- **L844 EN**: Continues the surrounding expression or declaration: `"%select{"`.
  **L844 CN**: 继续构造周围的表达式或声明：`"%select{"`。
- **L845 EN**: Continues the surrounding expression or declaration: `"%select{typedef|type alias}4 name %5|"`.
  **L845 CN**: 继续构造周围的表达式或声明：`"%select{typedef|type alias}4 name %5|"`。
- **L846 EN**: Continues the surrounding expression or declaration: `"%select{typedef|type alias}4 %5 with underlying type %6"`.
  **L846 CN**: 继续构造周围的表达式或声明：`"%select{typedef|type alias}4 %5 with underlying type %6"`。
- **L847 EN**: Adds a standalone statement or declaration: `"}3">;`.
  **L847 CN**: 添加一条独立语句或声明：`"}3">;`。
- **L848 EN**: Declares TableGen def record `note_module_odr_violation_typedef`.
  **L848 CN**: 声明 TableGen def 记录 `note_module_odr_violation_typedef`。
- **L849 EN**: Continues the surrounding expression or declaration: `"%select{"`.
  **L849 CN**: 继续构造周围的表达式或声明：`"%select{"`。
- **L850 EN**: Continues the surrounding expression or declaration: `"%select{typedef|type alias}2 name %3|"`.
  **L850 CN**: 继续构造周围的表达式或声明：`"%select{typedef|type alias}2 name %3|"`。
- **L851 EN**: Continues the surrounding expression or declaration: `"%select{typedef|type alias}2 %3 with different underlying type %4"`.
  **L851 CN**: 继续构造周围的表达式或声明：`"%select{typedef|type alias}2 %3 with different underlying type %4"`。
- **L852 EN**: Adds a standalone statement or declaration: `"}1">;`.
  **L852 CN**: 添加一条独立语句或声明：`"}1">;`。
- **L853 EN**: Blank line separating nearby declarations or logic blocks.
  **L853 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L854 EN**: Declares TableGen def record `err_module_odr_violation_variable`.
  **L854 CN**: 声明 TableGen def 记录 `err_module_odr_violation_variable`。
- **L855 EN**: Continues the surrounding expression or declaration: `"%q0 has different definitions in different modules; first difference is "`.
  **L855 CN**: 继续构造周围的表达式或声明：`"%q0 has different definitions in different modules; first difference is "`。
- **L856 EN**: Continues the surrounding expression or declaration: `"%select{definition in module '%2'|defined here}1 found "`.
  **L856 CN**: 继续构造周围的表达式或声明：`"%select{definition in module '%2'|defined here}1 found "`。
- **L857 EN**: Continues the surrounding expression or declaration: `"%select{"`.
  **L857 CN**: 继续构造周围的表达式或声明：`"%select{"`。
- **L858 EN**: Continues the surrounding expression or declaration: `"data member with name %4|"`.
  **L858 CN**: 继续构造周围的表达式或声明：`"data member with name %4|"`。
- **L859 EN**: Continues the surrounding expression or declaration: `"data member %4 with type %5|"`.
  **L859 CN**: 继续构造周围的表达式或声明：`"data member %4 with type %5|"`。
- **L860 EN**: Continues the surrounding expression or declaration: `"data member %4 with%select{out|}5 an initializer|"`.
  **L860 CN**: 继续构造周围的表达式或声明：`"data member %4 with%select{out|}5 an initializer|"`。
- **L861 EN**: Continues the surrounding expression or declaration: `"data member %4 with an initializer|"`.
  **L861 CN**: 继续构造周围的表达式或声明：`"data member %4 with an initializer|"`。
- **L862 EN**: Continues the surrounding expression or declaration: `"data member %4 %select{is constexpr|is not constexpr}5"`.
  **L862 CN**: 继续构造周围的表达式或声明：`"data member %4 %select{is constexpr|is not constexpr}5"`。
- **L863 EN**: Adds a standalone statement or declaration: `"}3">;`.
  **L863 CN**: 添加一条独立语句或声明：`"}3">;`。
- **L864 EN**: Declares TableGen def record `note_module_odr_violation_variable`.
  **L864 CN**: 声明 TableGen def 记录 `note_module_odr_violation_variable`。

### Lines 865-888

````tablegen
  "%select{"
  "data member with name %2|"
  "data member %2 with different type %3|"
  "data member %2 with%select{out|}3 an initializer|"
  "data member %2 with a different initializer|"
  "data member %2 %select{is constexpr|is not constexpr}3"
  "}1">;

def err_module_odr_violation_function : Error<
  "%q0 has different definitions in different modules; "
  "%select{definition in module '%2'|defined here}1 "
  "first difference is "
  "%select{"
  "return type is %4|"
  "%ordinal4 parameter with name %5|"
  "%ordinal4 parameter with type %5%select{| decayed from %7}6|"
  "%ordinal4 parameter with%select{out|}5 a default argument|"
  "%ordinal4 parameter with a default argument|"
  "function body"
  "}3">;

def note_module_odr_violation_function : Note<"but in '%0' found "
  "%select{"
  "different return type %2|"
````
- **L865 EN**: Continues the surrounding expression or declaration: `"%select{"`.
  **L865 CN**: 继续构造周围的表达式或声明：`"%select{"`。
- **L866 EN**: Continues the surrounding expression or declaration: `"data member with name %2|"`.
  **L866 CN**: 继续构造周围的表达式或声明：`"data member with name %2|"`。
- **L867 EN**: Continues the surrounding expression or declaration: `"data member %2 with different type %3|"`.
  **L867 CN**: 继续构造周围的表达式或声明：`"data member %2 with different type %3|"`。
- **L868 EN**: Continues the surrounding expression or declaration: `"data member %2 with%select{out|}3 an initializer|"`.
  **L868 CN**: 继续构造周围的表达式或声明：`"data member %2 with%select{out|}3 an initializer|"`。
- **L869 EN**: Continues the surrounding expression or declaration: `"data member %2 with a different initializer|"`.
  **L869 CN**: 继续构造周围的表达式或声明：`"data member %2 with a different initializer|"`。
- **L870 EN**: Continues the surrounding expression or declaration: `"data member %2 %select{is constexpr|is not constexpr}3"`.
  **L870 CN**: 继续构造周围的表达式或声明：`"data member %2 %select{is constexpr|is not constexpr}3"`。
- **L871 EN**: Adds a standalone statement or declaration: `"}1">;`.
  **L871 CN**: 添加一条独立语句或声明：`"}1">;`。
- **L872 EN**: Blank line separating nearby declarations or logic blocks.
  **L872 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L873 EN**: Declares TableGen def record `err_module_odr_violation_function`.
  **L873 CN**: 声明 TableGen def 记录 `err_module_odr_violation_function`。
- **L874 EN**: Continues the surrounding expression or declaration: `"%q0 has different definitions in different modules; "`.
  **L874 CN**: 继续构造周围的表达式或声明：`"%q0 has different definitions in different modules; "`。
- **L875 EN**: Continues the surrounding expression or declaration: `"%select{definition in module '%2'|defined here}1 "`.
  **L875 CN**: 继续构造周围的表达式或声明：`"%select{definition in module '%2'|defined here}1 "`。
- **L876 EN**: Continues the surrounding expression or declaration: `"first difference is "`.
  **L876 CN**: 继续构造周围的表达式或声明：`"first difference is "`。
- **L877 EN**: Continues the surrounding expression or declaration: `"%select{"`.
  **L877 CN**: 继续构造周围的表达式或声明：`"%select{"`。
- **L878 EN**: Continues the surrounding expression or declaration: `"return type is %4|"`.
  **L878 CN**: 继续构造周围的表达式或声明：`"return type is %4|"`。
- **L879 EN**: Continues the surrounding expression or declaration: `"%ordinal4 parameter with name %5|"`.
  **L879 CN**: 继续构造周围的表达式或声明：`"%ordinal4 parameter with name %5|"`。
- **L880 EN**: Continues the surrounding expression or declaration: `"%ordinal4 parameter with type %5%select{| decayed from %7}6|"`.
  **L880 CN**: 继续构造周围的表达式或声明：`"%ordinal4 parameter with type %5%select{| decayed from %7}6|"`。
- **L881 EN**: Continues the surrounding expression or declaration: `"%ordinal4 parameter with%select{out|}5 a default argument|"`.
  **L881 CN**: 继续构造周围的表达式或声明：`"%ordinal4 parameter with%select{out|}5 a default argument|"`。
- **L882 EN**: Continues the surrounding expression or declaration: `"%ordinal4 parameter with a default argument|"`.
  **L882 CN**: 继续构造周围的表达式或声明：`"%ordinal4 parameter with a default argument|"`。
- **L883 EN**: Continues the surrounding expression or declaration: `"function body"`.
  **L883 CN**: 继续构造周围的表达式或声明：`"function body"`。
- **L884 EN**: Adds a standalone statement or declaration: `"}3">;`.
  **L884 CN**: 添加一条独立语句或声明：`"}3">;`。
- **L885 EN**: Blank line separating nearby declarations or logic blocks.
  **L885 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L886 EN**: Declares TableGen def record `note_module_odr_violation_function`.
  **L886 CN**: 声明 TableGen def 记录 `note_module_odr_violation_function`。
- **L887 EN**: Continues the surrounding expression or declaration: `"%select{"`.
  **L887 CN**: 继续构造周围的表达式或声明：`"%select{"`。
- **L888 EN**: Continues the surrounding expression or declaration: `"different return type %2|"`.
  **L888 CN**: 继续构造周围的表达式或声明：`"different return type %2|"`。

### Lines 889-912

````tablegen
  "%ordinal2 parameter with name %3|"
  "%ordinal2 parameter with type %3%select{| decayed from %5}4|"
  "%ordinal2 parameter with%select{out|}3 a default argument|"
  "%ordinal2 parameter with a different default argument|"
  "a different body"
  "}1">;

def err_module_odr_violation_enum : Error<
  "%q0 has different definitions in different modules; "
  "%select{definition in module '%2'|defined here}1 "
  "first difference is "
  "%select{"
  "enum that is %select{not scoped|scoped}4|"
  "enum scoped with keyword %select{struct|class}4|"
  "enum %select{without|with}4 specified type|"
  "enum with specified type %4|"
  "enum with %4 element%s4|"
  "%ordinal4 element has name %5|"
  "%ordinal4 element %5 %select{has|does not have}6 an initializer|"
  "%ordinal4 element %5 has an initializer|"
  "}3">;

def note_module_odr_violation_enum : Note<"but in '%0' found "
  "%select{"
````
- **L889 EN**: Continues the surrounding expression or declaration: `"%ordinal2 parameter with name %3|"`.
  **L889 CN**: 继续构造周围的表达式或声明：`"%ordinal2 parameter with name %3|"`。
- **L890 EN**: Continues the surrounding expression or declaration: `"%ordinal2 parameter with type %3%select{| decayed from %5}4|"`.
  **L890 CN**: 继续构造周围的表达式或声明：`"%ordinal2 parameter with type %3%select{| decayed from %5}4|"`。
- **L891 EN**: Continues the surrounding expression or declaration: `"%ordinal2 parameter with%select{out|}3 a default argument|"`.
  **L891 CN**: 继续构造周围的表达式或声明：`"%ordinal2 parameter with%select{out|}3 a default argument|"`。
- **L892 EN**: Continues the surrounding expression or declaration: `"%ordinal2 parameter with a different default argument|"`.
  **L892 CN**: 继续构造周围的表达式或声明：`"%ordinal2 parameter with a different default argument|"`。
- **L893 EN**: Continues the surrounding expression or declaration: `"a different body"`.
  **L893 CN**: 继续构造周围的表达式或声明：`"a different body"`。
- **L894 EN**: Adds a standalone statement or declaration: `"}1">;`.
  **L894 CN**: 添加一条独立语句或声明：`"}1">;`。
- **L895 EN**: Blank line separating nearby declarations or logic blocks.
  **L895 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L896 EN**: Declares TableGen def record `err_module_odr_violation_enum`.
  **L896 CN**: 声明 TableGen def 记录 `err_module_odr_violation_enum`。
- **L897 EN**: Continues the surrounding expression or declaration: `"%q0 has different definitions in different modules; "`.
  **L897 CN**: 继续构造周围的表达式或声明：`"%q0 has different definitions in different modules; "`。
- **L898 EN**: Continues the surrounding expression or declaration: `"%select{definition in module '%2'|defined here}1 "`.
  **L898 CN**: 继续构造周围的表达式或声明：`"%select{definition in module '%2'|defined here}1 "`。
- **L899 EN**: Continues the surrounding expression or declaration: `"first difference is "`.
  **L899 CN**: 继续构造周围的表达式或声明：`"first difference is "`。
- **L900 EN**: Continues the surrounding expression or declaration: `"%select{"`.
  **L900 CN**: 继续构造周围的表达式或声明：`"%select{"`。
- **L901 EN**: Continues the surrounding expression or declaration: `"enum that is %select{not scoped|scoped}4|"`.
  **L901 CN**: 继续构造周围的表达式或声明：`"enum that is %select{not scoped|scoped}4|"`。
- **L902 EN**: Continues the surrounding expression or declaration: `"enum scoped with keyword %select{struct|class}4|"`.
  **L902 CN**: 继续构造周围的表达式或声明：`"enum scoped with keyword %select{struct|class}4|"`。
- **L903 EN**: Continues the surrounding expression or declaration: `"enum %select{without|with}4 specified type|"`.
  **L903 CN**: 继续构造周围的表达式或声明：`"enum %select{without|with}4 specified type|"`。
- **L904 EN**: Continues the surrounding expression or declaration: `"enum with specified type %4|"`.
  **L904 CN**: 继续构造周围的表达式或声明：`"enum with specified type %4|"`。
- **L905 EN**: Continues the surrounding expression or declaration: `"enum with %4 element%s4|"`.
  **L905 CN**: 继续构造周围的表达式或声明：`"enum with %4 element%s4|"`。
- **L906 EN**: Continues the surrounding expression or declaration: `"%ordinal4 element has name %5|"`.
  **L906 CN**: 继续构造周围的表达式或声明：`"%ordinal4 element has name %5|"`。
- **L907 EN**: Continues the surrounding expression or declaration: `"%ordinal4 element %5 %select{has|does not have}6 an initializer|"`.
  **L907 CN**: 继续构造周围的表达式或声明：`"%ordinal4 element %5 %select{has|does not have}6 an initializer|"`。
- **L908 EN**: Continues the surrounding expression or declaration: `"%ordinal4 element %5 has an initializer|"`.
  **L908 CN**: 继续构造周围的表达式或声明：`"%ordinal4 element %5 has an initializer|"`。
- **L909 EN**: Adds a standalone statement or declaration: `"}3">;`.
  **L909 CN**: 添加一条独立语句或声明：`"}3">;`。
- **L910 EN**: Blank line separating nearby declarations or logic blocks.
  **L910 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L911 EN**: Declares TableGen def record `note_module_odr_violation_enum`.
  **L911 CN**: 声明 TableGen def 记录 `note_module_odr_violation_enum`。
- **L912 EN**: Continues the surrounding expression or declaration: `"%select{"`.
  **L912 CN**: 继续构造周围的表达式或声明：`"%select{"`。

### Lines 913-936

````tablegen
  "enum that is %select{not scoped|scoped}2|"
  "enum scoped with keyword %select{struct|class}2|"
  "enum %select{without|with}2 specified type|"
  "enum with specified type %2|"
  "enum with %2 element%s2|"
  "%ordinal2 element has name %3|"
  "%ordinal2 element %3 %select{has|does not have}4 an initializer|"
  "%ordinal2 element %3 has different initializer|"
  "}1">;

def err_module_odr_violation_referenced_protocols : Error <
  "%q0 has different definitions in different modules; first difference is "
  "%select{definition in module '%2'|defined here}1 found "
  "%select{"
  "%4 referenced %plural{1:protocol|:protocols}4|"
  "%ordinal4 referenced protocol with name %5"
  "}3">;
def note_module_odr_violation_referenced_protocols : Note <
  "but in %select{'%1'|definition here}0 found "
  "%select{"
  "%3 referenced %plural{1:protocol|:protocols}3|"
  "%ordinal3 referenced protocol with different name %4"
  "}2">;

````
- **L913 EN**: Continues the surrounding expression or declaration: `"enum that is %select{not scoped|scoped}2|"`.
  **L913 CN**: 继续构造周围的表达式或声明：`"enum that is %select{not scoped|scoped}2|"`。
- **L914 EN**: Continues the surrounding expression or declaration: `"enum scoped with keyword %select{struct|class}2|"`.
  **L914 CN**: 继续构造周围的表达式或声明：`"enum scoped with keyword %select{struct|class}2|"`。
- **L915 EN**: Continues the surrounding expression or declaration: `"enum %select{without|with}2 specified type|"`.
  **L915 CN**: 继续构造周围的表达式或声明：`"enum %select{without|with}2 specified type|"`。
- **L916 EN**: Continues the surrounding expression or declaration: `"enum with specified type %2|"`.
  **L916 CN**: 继续构造周围的表达式或声明：`"enum with specified type %2|"`。
- **L917 EN**: Continues the surrounding expression or declaration: `"enum with %2 element%s2|"`.
  **L917 CN**: 继续构造周围的表达式或声明：`"enum with %2 element%s2|"`。
- **L918 EN**: Continues the surrounding expression or declaration: `"%ordinal2 element has name %3|"`.
  **L918 CN**: 继续构造周围的表达式或声明：`"%ordinal2 element has name %3|"`。
- **L919 EN**: Continues the surrounding expression or declaration: `"%ordinal2 element %3 %select{has|does not have}4 an initializer|"`.
  **L919 CN**: 继续构造周围的表达式或声明：`"%ordinal2 element %3 %select{has|does not have}4 an initializer|"`。
- **L920 EN**: Continues the surrounding expression or declaration: `"%ordinal2 element %3 has different initializer|"`.
  **L920 CN**: 继续构造周围的表达式或声明：`"%ordinal2 element %3 has different initializer|"`。
- **L921 EN**: Adds a standalone statement or declaration: `"}1">;`.
  **L921 CN**: 添加一条独立语句或声明：`"}1">;`。
- **L922 EN**: Blank line separating nearby declarations or logic blocks.
  **L922 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L923 EN**: Declares TableGen def record `err_module_odr_violation_referenced_protocols`.
  **L923 CN**: 声明 TableGen def 记录 `err_module_odr_violation_referenced_protocols`。
- **L924 EN**: Continues the surrounding expression or declaration: `"%q0 has different definitions in different modules; first difference is "`.
  **L924 CN**: 继续构造周围的表达式或声明：`"%q0 has different definitions in different modules; first difference is "`。
- **L925 EN**: Continues the surrounding expression or declaration: `"%select{definition in module '%2'|defined here}1 found "`.
  **L925 CN**: 继续构造周围的表达式或声明：`"%select{definition in module '%2'|defined here}1 found "`。
- **L926 EN**: Continues the surrounding expression or declaration: `"%select{"`.
  **L926 CN**: 继续构造周围的表达式或声明：`"%select{"`。
- **L927 EN**: Continues the surrounding expression or declaration: `"%4 referenced %plural{1:protocol|:protocols}4|"`.
  **L927 CN**: 继续构造周围的表达式或声明：`"%4 referenced %plural{1:protocol|:protocols}4|"`。
- **L928 EN**: Continues the surrounding expression or declaration: `"%ordinal4 referenced protocol with name %5"`.
  **L928 CN**: 继续构造周围的表达式或声明：`"%ordinal4 referenced protocol with name %5"`。
- **L929 EN**: Adds a standalone statement or declaration: `"}3">;`.
  **L929 CN**: 添加一条独立语句或声明：`"}3">;`。
- **L930 EN**: Declares TableGen def record `note_module_odr_violation_referenced_protocols`.
  **L930 CN**: 声明 TableGen def 记录 `note_module_odr_violation_referenced_protocols`。
- **L931 EN**: Continues the surrounding expression or declaration: `"but in %select{'%1'|definition here}0 found "`.
  **L931 CN**: 继续构造周围的表达式或声明：`"but in %select{'%1'|definition here}0 found "`。
- **L932 EN**: Continues the surrounding expression or declaration: `"%select{"`.
  **L932 CN**: 继续构造周围的表达式或声明：`"%select{"`。
- **L933 EN**: Continues the surrounding expression or declaration: `"%3 referenced %plural{1:protocol|:protocols}3|"`.
  **L933 CN**: 继续构造周围的表达式或声明：`"%3 referenced %plural{1:protocol|:protocols}3|"`。
- **L934 EN**: Continues the surrounding expression or declaration: `"%ordinal3 referenced protocol with different name %4"`.
  **L934 CN**: 继续构造周围的表达式或声明：`"%ordinal3 referenced protocol with different name %4"`。
- **L935 EN**: Adds a standalone statement or declaration: `"}2">;`.
  **L935 CN**: 添加一条独立语句或声明：`"}2">;`。
- **L936 EN**: Blank line separating nearby declarations or logic blocks.
  **L936 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 937-960

````tablegen
def err_module_odr_violation_objc_method : Error<
  "%q0 has different definitions in different modules; first difference is "
  "%select{definition in module '%2'|defined here}1 found "
  "%select{"
  "method %4 with return type %5|"
  "%select{class|instance}5 method %4|"
  "%select{no|'required'|'optional'}4 method control|"
  "method %4 with %select{no designated initializer|designated initializer}5|"
  "%select{regular|direct}5 method %4|"
  "method %4"
  "}3">;
def note_module_odr_violation_objc_method : Note<
  "but in %select{'%1'|definition here}0 found "
  "%select{"
  "method %3 with different return type %4|"
  "method %3 as %select{class|instance}4 method|"
  "%select{no|'required'|'optional'}3 method control|"
  "method %3 with %select{no designated initializer|designated initializer}4|"
  "%select{regular|direct}4 method %3|"
  "different method %3"
  "}2">;

def err_module_odr_violation_method_params : Error<
  "%q0 has different definitions in different modules; first difference is "
````
- **L937 EN**: Declares TableGen def record `err_module_odr_violation_objc_method`.
  **L937 CN**: 声明 TableGen def 记录 `err_module_odr_violation_objc_method`。
- **L938 EN**: Continues the surrounding expression or declaration: `"%q0 has different definitions in different modules; first difference is "`.
  **L938 CN**: 继续构造周围的表达式或声明：`"%q0 has different definitions in different modules; first difference is "`。
- **L939 EN**: Continues the surrounding expression or declaration: `"%select{definition in module '%2'|defined here}1 found "`.
  **L939 CN**: 继续构造周围的表达式或声明：`"%select{definition in module '%2'|defined here}1 found "`。
- **L940 EN**: Continues the surrounding expression or declaration: `"%select{"`.
  **L940 CN**: 继续构造周围的表达式或声明：`"%select{"`。
- **L941 EN**: Continues the surrounding expression or declaration: `"method %4 with return type %5|"`.
  **L941 CN**: 继续构造周围的表达式或声明：`"method %4 with return type %5|"`。
- **L942 EN**: Continues the surrounding expression or declaration: `"%select{class|instance}5 method %4|"`.
  **L942 CN**: 继续构造周围的表达式或声明：`"%select{class|instance}5 method %4|"`。
- **L943 EN**: Continues the surrounding expression or declaration: `"%select{no|'required'|'optional'}4 method control|"`.
  **L943 CN**: 继续构造周围的表达式或声明：`"%select{no|'required'|'optional'}4 method control|"`。
- **L944 EN**: Continues the surrounding expression or declaration: `"method %4 with %select{no designated initializer|designated initializer}5|"`.
  **L944 CN**: 继续构造周围的表达式或声明：`"method %4 with %select{no designated initializer|designated initializer}5|"`。
- **L945 EN**: Continues the surrounding expression or declaration: `"%select{regular|direct}5 method %4|"`.
  **L945 CN**: 继续构造周围的表达式或声明：`"%select{regular|direct}5 method %4|"`。
- **L946 EN**: Continues the surrounding expression or declaration: `"method %4"`.
  **L946 CN**: 继续构造周围的表达式或声明：`"method %4"`。
- **L947 EN**: Adds a standalone statement or declaration: `"}3">;`.
  **L947 CN**: 添加一条独立语句或声明：`"}3">;`。
- **L948 EN**: Declares TableGen def record `note_module_odr_violation_objc_method`.
  **L948 CN**: 声明 TableGen def 记录 `note_module_odr_violation_objc_method`。
- **L949 EN**: Continues the surrounding expression or declaration: `"but in %select{'%1'|definition here}0 found "`.
  **L949 CN**: 继续构造周围的表达式或声明：`"but in %select{'%1'|definition here}0 found "`。
- **L950 EN**: Continues the surrounding expression or declaration: `"%select{"`.
  **L950 CN**: 继续构造周围的表达式或声明：`"%select{"`。
- **L951 EN**: Continues the surrounding expression or declaration: `"method %3 with different return type %4|"`.
  **L951 CN**: 继续构造周围的表达式或声明：`"method %3 with different return type %4|"`。
- **L952 EN**: Continues the surrounding expression or declaration: `"method %3 as %select{class|instance}4 method|"`.
  **L952 CN**: 继续构造周围的表达式或声明：`"method %3 as %select{class|instance}4 method|"`。
- **L953 EN**: Continues the surrounding expression or declaration: `"%select{no|'required'|'optional'}3 method control|"`.
  **L953 CN**: 继续构造周围的表达式或声明：`"%select{no|'required'|'optional'}3 method control|"`。
- **L954 EN**: Continues the surrounding expression or declaration: `"method %3 with %select{no designated initializer|designated initializer}4|"`.
  **L954 CN**: 继续构造周围的表达式或声明：`"method %3 with %select{no designated initializer|designated initializer}4|"`。
- **L955 EN**: Continues the surrounding expression or declaration: `"%select{regular|direct}4 method %3|"`.
  **L955 CN**: 继续构造周围的表达式或声明：`"%select{regular|direct}4 method %3|"`。
- **L956 EN**: Continues the surrounding expression or declaration: `"different method %3"`.
  **L956 CN**: 继续构造周围的表达式或声明：`"different method %3"`。
- **L957 EN**: Adds a standalone statement or declaration: `"}2">;`.
  **L957 CN**: 添加一条独立语句或声明：`"}2">;`。
- **L958 EN**: Blank line separating nearby declarations or logic blocks.
  **L958 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L959 EN**: Declares TableGen def record `err_module_odr_violation_method_params`.
  **L959 CN**: 声明 TableGen def 记录 `err_module_odr_violation_method_params`。
- **L960 EN**: Continues the surrounding expression or declaration: `"%q0 has different definitions in different modules; first difference is "`.
  **L960 CN**: 继续构造周围的表达式或声明：`"%q0 has different definitions in different modules; first difference is "`。

### Lines 961-984

````tablegen
  "%select{definition in module '%2'|defined here}1 found "
  "%select{"
  "%select{method %5|constructor|destructor}4 "
    "that has %6 parameter%s6|"
  "%select{method %5|constructor|destructor}4 "
    "with %ordinal6 parameter of type %7%select{| decayed from %9}8|"
  "%select{method %5|constructor|destructor}4 "
    "with %ordinal6 parameter named %7"
  "}3">;
def note_module_odr_violation_method_params : Note<
  "but in %select{'%1'|definition here}0 found "
  "%select{"
  "%select{method %4|constructor|destructor}3 "
    "that has %5 parameter%s5|"
  "%select{method %4|constructor|destructor}3 "
    "with %ordinal5 parameter of type %6%select{| decayed from %8}7|"
  "%select{method %4|constructor|destructor}3 "
    "with %ordinal5 parameter named %6"
  "}2">;

def err_module_odr_violation_objc_property : Error<
  "%q0 has different definitions in different modules; first difference is "
  "%select{definition in module '%2'|defined here}1 found "
  "%select{"
````
- **L961 EN**: Continues the surrounding expression or declaration: `"%select{definition in module '%2'|defined here}1 found "`.
  **L961 CN**: 继续构造周围的表达式或声明：`"%select{definition in module '%2'|defined here}1 found "`。
- **L962 EN**: Continues the surrounding expression or declaration: `"%select{"`.
  **L962 CN**: 继续构造周围的表达式或声明：`"%select{"`。
- **L963 EN**: Continues the surrounding expression or declaration: `"%select{method %5|constructor|destructor}4 "`.
  **L963 CN**: 继续构造周围的表达式或声明：`"%select{method %5|constructor|destructor}4 "`。
- **L964 EN**: Continues the surrounding expression or declaration: `"that has %6 parameter%s6|"`.
  **L964 CN**: 继续构造周围的表达式或声明：`"that has %6 parameter%s6|"`。
- **L965 EN**: Continues the surrounding expression or declaration: `"%select{method %5|constructor|destructor}4 "`.
  **L965 CN**: 继续构造周围的表达式或声明：`"%select{method %5|constructor|destructor}4 "`。
- **L966 EN**: Continues the surrounding expression or declaration: `"with %ordinal6 parameter of type %7%select{| decayed from %9}8|"`.
  **L966 CN**: 继续构造周围的表达式或声明：`"with %ordinal6 parameter of type %7%select{| decayed from %9}8|"`。
- **L967 EN**: Continues the surrounding expression or declaration: `"%select{method %5|constructor|destructor}4 "`.
  **L967 CN**: 继续构造周围的表达式或声明：`"%select{method %5|constructor|destructor}4 "`。
- **L968 EN**: Continues the surrounding expression or declaration: `"with %ordinal6 parameter named %7"`.
  **L968 CN**: 继续构造周围的表达式或声明：`"with %ordinal6 parameter named %7"`。
- **L969 EN**: Adds a standalone statement or declaration: `"}3">;`.
  **L969 CN**: 添加一条独立语句或声明：`"}3">;`。
- **L970 EN**: Declares TableGen def record `note_module_odr_violation_method_params`.
  **L970 CN**: 声明 TableGen def 记录 `note_module_odr_violation_method_params`。
- **L971 EN**: Continues the surrounding expression or declaration: `"but in %select{'%1'|definition here}0 found "`.
  **L971 CN**: 继续构造周围的表达式或声明：`"but in %select{'%1'|definition here}0 found "`。
- **L972 EN**: Continues the surrounding expression or declaration: `"%select{"`.
  **L972 CN**: 继续构造周围的表达式或声明：`"%select{"`。
- **L973 EN**: Continues the surrounding expression or declaration: `"%select{method %4|constructor|destructor}3 "`.
  **L973 CN**: 继续构造周围的表达式或声明：`"%select{method %4|constructor|destructor}3 "`。
- **L974 EN**: Continues the surrounding expression or declaration: `"that has %5 parameter%s5|"`.
  **L974 CN**: 继续构造周围的表达式或声明：`"that has %5 parameter%s5|"`。
- **L975 EN**: Continues the surrounding expression or declaration: `"%select{method %4|constructor|destructor}3 "`.
  **L975 CN**: 继续构造周围的表达式或声明：`"%select{method %4|constructor|destructor}3 "`。
- **L976 EN**: Continues the surrounding expression or declaration: `"with %ordinal5 parameter of type %6%select{| decayed from %8}7|"`.
  **L976 CN**: 继续构造周围的表达式或声明：`"with %ordinal5 parameter of type %6%select{| decayed from %8}7|"`。
- **L977 EN**: Continues the surrounding expression or declaration: `"%select{method %4|constructor|destructor}3 "`.
  **L977 CN**: 继续构造周围的表达式或声明：`"%select{method %4|constructor|destructor}3 "`。
- **L978 EN**: Continues the surrounding expression or declaration: `"with %ordinal5 parameter named %6"`.
  **L978 CN**: 继续构造周围的表达式或声明：`"with %ordinal5 parameter named %6"`。
- **L979 EN**: Adds a standalone statement or declaration: `"}2">;`.
  **L979 CN**: 添加一条独立语句或声明：`"}2">;`。
- **L980 EN**: Blank line separating nearby declarations or logic blocks.
  **L980 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L981 EN**: Declares TableGen def record `err_module_odr_violation_objc_property`.
  **L981 CN**: 声明 TableGen def 记录 `err_module_odr_violation_objc_property`。
- **L982 EN**: Continues the surrounding expression or declaration: `"%q0 has different definitions in different modules; first difference is "`.
  **L982 CN**: 继续构造周围的表达式或声明：`"%q0 has different definitions in different modules; first difference is "`。
- **L983 EN**: Continues the surrounding expression or declaration: `"%select{definition in module '%2'|defined here}1 found "`.
  **L983 CN**: 继续构造周围的表达式或声明：`"%select{definition in module '%2'|defined here}1 found "`。
- **L984 EN**: Continues the surrounding expression or declaration: `"%select{"`.
  **L984 CN**: 继续构造周围的表达式或声明：`"%select{"`。

### Lines 985-1008

````tablegen
  "property %4|"
  "property %4 with type %5|"
  "%select{no|'required'|'optional'}4 property control|"
  "property %4 with %select{default |}6'%select{none|readonly|getter|assign|"
    "readwrite|retain|copy|nonatomic|setter|atomic|weak|strong|"
    "unsafe_unretained|nullability|null_resettable|class|direct}5' attribute"
  "}3">;
def note_module_odr_violation_objc_property : Note<
  "but in %select{'%1'|definition here}0 found "
  "%select{"
  "property %3|"
  "property %3 with type %4|"
  "%select{no|'required'|'optional'}3 property control|"
  "property %3 with different '%select{none|readonly|getter|assign|"
    "readwrite|retain|copy|nonatomic|setter|atomic|weak|strong|"
    "unsafe_unretained|nullability|null_resettable|class|direct}4' attribute"
  "}2">;

def err_module_odr_violation_mismatch_decl_unknown : Error<
  "%q0 %select{with definition in module '%2'|defined here}1 has different "
  "definitions in different modules; first difference is this "
  "%select{||||static assert|field|method|type alias|typedef|data member|"
  "friend declaration|function template|method|instance variable|"
  "property|unexpected decl}3">;
````
- **L985 EN**: Continues the surrounding expression or declaration: `"property %4|"`.
  **L985 CN**: 继续构造周围的表达式或声明：`"property %4|"`。
- **L986 EN**: Continues the surrounding expression or declaration: `"property %4 with type %5|"`.
  **L986 CN**: 继续构造周围的表达式或声明：`"property %4 with type %5|"`。
- **L987 EN**: Continues the surrounding expression or declaration: `"%select{no|'required'|'optional'}4 property control|"`.
  **L987 CN**: 继续构造周围的表达式或声明：`"%select{no|'required'|'optional'}4 property control|"`。
- **L988 EN**: Continues the surrounding expression or declaration: `"property %4 with %select{default |}6'%select{none|readonly|getter|assign|"`.
  **L988 CN**: 继续构造周围的表达式或声明：`"property %4 with %select{default |}6'%select{none|readonly|getter|assign|"`。
- **L989 EN**: Continues the surrounding expression or declaration: `"readwrite|retain|copy|nonatomic|setter|atomic|weak|strong|"`.
  **L989 CN**: 继续构造周围的表达式或声明：`"readwrite|retain|copy|nonatomic|setter|atomic|weak|strong|"`。
- **L990 EN**: Continues the surrounding expression or declaration: `"unsafe_unretained|nullability|null_resettable|class|direct}5' attribute"`.
  **L990 CN**: 继续构造周围的表达式或声明：`"unsafe_unretained|nullability|null_resettable|class|direct}5' attribute"`。
- **L991 EN**: Adds a standalone statement or declaration: `"}3">;`.
  **L991 CN**: 添加一条独立语句或声明：`"}3">;`。
- **L992 EN**: Declares TableGen def record `note_module_odr_violation_objc_property`.
  **L992 CN**: 声明 TableGen def 记录 `note_module_odr_violation_objc_property`。
- **L993 EN**: Continues the surrounding expression or declaration: `"but in %select{'%1'|definition here}0 found "`.
  **L993 CN**: 继续构造周围的表达式或声明：`"but in %select{'%1'|definition here}0 found "`。
- **L994 EN**: Continues the surrounding expression or declaration: `"%select{"`.
  **L994 CN**: 继续构造周围的表达式或声明：`"%select{"`。
- **L995 EN**: Continues the surrounding expression or declaration: `"property %3|"`.
  **L995 CN**: 继续构造周围的表达式或声明：`"property %3|"`。
- **L996 EN**: Continues the surrounding expression or declaration: `"property %3 with type %4|"`.
  **L996 CN**: 继续构造周围的表达式或声明：`"property %3 with type %4|"`。
- **L997 EN**: Continues the surrounding expression or declaration: `"%select{no|'required'|'optional'}3 property control|"`.
  **L997 CN**: 继续构造周围的表达式或声明：`"%select{no|'required'|'optional'}3 property control|"`。
- **L998 EN**: Continues the surrounding expression or declaration: `"property %3 with different '%select{none|readonly|getter|assign|"`.
  **L998 CN**: 继续构造周围的表达式或声明：`"property %3 with different '%select{none|readonly|getter|assign|"`。
- **L999 EN**: Continues the surrounding expression or declaration: `"readwrite|retain|copy|nonatomic|setter|atomic|weak|strong|"`.
  **L999 CN**: 继续构造周围的表达式或声明：`"readwrite|retain|copy|nonatomic|setter|atomic|weak|strong|"`。
- **L1000 EN**: Continues the surrounding expression or declaration: `"unsafe_unretained|nullability|null_resettable|class|direct}4' attribute"`.
  **L1000 CN**: 继续构造周围的表达式或声明：`"unsafe_unretained|nullability|null_resettable|class|direct}4' attribute"`。
- **L1001 EN**: Adds a standalone statement or declaration: `"}2">;`.
  **L1001 CN**: 添加一条独立语句或声明：`"}2">;`。
- **L1002 EN**: Blank line separating nearby declarations or logic blocks.
  **L1002 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1003 EN**: Declares TableGen def record `err_module_odr_violation_mismatch_decl_unknown`.
  **L1003 CN**: 声明 TableGen def 记录 `err_module_odr_violation_mismatch_decl_unknown`。
- **L1004 EN**: Continues the surrounding expression or declaration: `"%q0 %select{with definition in module '%2'|defined here}1 has different "`.
  **L1004 CN**: 继续构造周围的表达式或声明：`"%q0 %select{with definition in module '%2'|defined here}1 has different "`。
- **L1005 EN**: Continues the surrounding expression or declaration: `"definitions in different modules; first difference is this "`.
  **L1005 CN**: 继续构造周围的表达式或声明：`"definitions in different modules; first difference is this "`。
- **L1006 EN**: Continues the surrounding expression or declaration: `"%select{||||static assert|field|method|type alias|typedef|data member|"`.
  **L1006 CN**: 继续构造周围的表达式或声明：`"%select{||||static assert|field|method|type alias|typedef|data member|"`。
- **L1007 EN**: Continues the surrounding expression or declaration: `"friend declaration|function template|method|instance variable|"`.
  **L1007 CN**: 继续构造周围的表达式或声明：`"friend declaration|function template|method|instance variable|"`。
- **L1008 EN**: Adds a standalone statement or declaration: `"property|unexpected decl}3">;`.
  **L1008 CN**: 添加一条独立语句或声明：`"property|unexpected decl}3">;`。

### Lines 1009-1032

````tablegen
def note_module_odr_violation_mismatch_decl_unknown : Note<
  "but in %select{'%1'|definition here}0 found "
  "%select{||||different static assert|different field|different method|"
  "different type alias|different typedef|different data member|"
  "different friend declaration|different function template|different method|"
  "different instance variable|different property|another unexpected decl}2">;

def err_struct_too_large : Error<
  "structure '%0' is too large, which exceeds maximum allowed size of %1 bytes">;

def remark_sanitize_address_insert_extra_padding_accepted : Remark<
    "-fsanitize-address-field-padding applied to %0">, ShowInSystemHeader,
    InGroup<SanitizeAddressRemarks>;
def remark_sanitize_address_insert_extra_padding_rejected : Remark<
    "-fsanitize-address-field-padding ignored for %0 because it "
    "%select{is not C++|is packed|is a union|is trivially copyable|"
    "has trivial destructor|is standard layout|is in a ignorelisted file|"
    "is ignorelisted}1">, ShowInSystemHeader,
    InGroup<SanitizeAddressRemarks>;

def warn_npot_ms_struct : Warning<
  "ms_struct may not produce Microsoft-compatible layouts with fundamental "
  "data types with sizes that aren't a power of two">,
  DefaultError, InGroup<IncompatibleMSStruct>;
````
- **L1009 EN**: Declares TableGen def record `note_module_odr_violation_mismatch_decl_unknown`.
  **L1009 CN**: 声明 TableGen def 记录 `note_module_odr_violation_mismatch_decl_unknown`。
- **L1010 EN**: Continues the surrounding expression or declaration: `"but in %select{'%1'|definition here}0 found "`.
  **L1010 CN**: 继续构造周围的表达式或声明：`"but in %select{'%1'|definition here}0 found "`。
- **L1011 EN**: Continues the surrounding expression or declaration: `"%select{||||different static assert|different field|different method|"`.
  **L1011 CN**: 继续构造周围的表达式或声明：`"%select{||||different static assert|different field|different method|"`。
- **L1012 EN**: Continues the surrounding expression or declaration: `"different type alias|different typedef|different data member|"`.
  **L1012 CN**: 继续构造周围的表达式或声明：`"different type alias|different typedef|different data member|"`。
- **L1013 EN**: Continues the surrounding expression or declaration: `"different friend declaration|different function template|different method|"`.
  **L1013 CN**: 继续构造周围的表达式或声明：`"different friend declaration|different function template|different method|"`。
- **L1014 EN**: Adds a standalone statement or declaration: `"different instance variable|different property|another unexpected decl}2">;`.
  **L1014 CN**: 添加一条独立语句或声明：`"different instance variable|different property|another unexpected decl}2">;`。
- **L1015 EN**: Blank line separating nearby declarations or logic blocks.
  **L1015 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1016 EN**: Declares TableGen def record `err_struct_too_large`.
  **L1016 CN**: 声明 TableGen def 记录 `err_struct_too_large`。
- **L1017 EN**: Adds a standalone statement or declaration: `"structure '%0' is too large, which exceeds maximum allowed size of %1 bytes">;`.
  **L1017 CN**: 添加一条独立语句或声明：`"structure '%0' is too large, which exceeds maximum allowed size of %1 bytes">;`。
- **L1018 EN**: Blank line separating nearby declarations or logic blocks.
  **L1018 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1019 EN**: Declares TableGen def record `remark_sanitize_address_insert_extra_padding_accepted`.
  **L1019 CN**: 声明 TableGen def 记录 `remark_sanitize_address_insert_extra_padding_accepted`。
- **L1020 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"-fsanitize-address-field-padding applied to %0">, ShowInSystemHeader,`.
  **L1020 CN**: 继续一个多行参数列表、初始化器或聚合项：`"-fsanitize-address-field-padding applied to %0">, ShowInSystemHeader,`。
- **L1021 EN**: Adds a standalone statement or declaration: `InGroup<SanitizeAddressRemarks>;`.
  **L1021 CN**: 添加一条独立语句或声明：`InGroup<SanitizeAddressRemarks>;`。
- **L1022 EN**: Declares TableGen def record `remark_sanitize_address_insert_extra_padding_rejected`.
  **L1022 CN**: 声明 TableGen def 记录 `remark_sanitize_address_insert_extra_padding_rejected`。
- **L1023 EN**: Continues the surrounding expression or declaration: `"-fsanitize-address-field-padding ignored for %0 because it "`.
  **L1023 CN**: 继续构造周围的表达式或声明：`"-fsanitize-address-field-padding ignored for %0 because it "`。
- **L1024 EN**: Continues the surrounding expression or declaration: `"%select{is not C++|is packed|is a union|is trivially copyable|"`.
  **L1024 CN**: 继续构造周围的表达式或声明：`"%select{is not C++|is packed|is a union|is trivially copyable|"`。
- **L1025 EN**: Continues the surrounding expression or declaration: `"has trivial destructor|is standard layout|is in a ignorelisted file|"`.
  **L1025 CN**: 继续构造周围的表达式或声明：`"has trivial destructor|is standard layout|is in a ignorelisted file|"`。
- **L1026 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"is ignorelisted}1">, ShowInSystemHeader,`.
  **L1026 CN**: 继续一个多行参数列表、初始化器或聚合项：`"is ignorelisted}1">, ShowInSystemHeader,`。
- **L1027 EN**: Adds a standalone statement or declaration: `InGroup<SanitizeAddressRemarks>;`.
  **L1027 CN**: 添加一条独立语句或声明：`InGroup<SanitizeAddressRemarks>;`。
- **L1028 EN**: Blank line separating nearby declarations or logic blocks.
  **L1028 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1029 EN**: Declares TableGen def record `warn_npot_ms_struct`.
  **L1029 CN**: 声明 TableGen def 记录 `warn_npot_ms_struct`。
- **L1030 EN**: Continues the surrounding expression or declaration: `"ms_struct may not produce Microsoft-compatible layouts with fundamental "`.
  **L1030 CN**: 继续构造周围的表达式或声明：`"ms_struct may not produce Microsoft-compatible layouts with fundamental "`。
- **L1031 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"data types with sizes that aren't a power of two">,`.
  **L1031 CN**: 继续一个多行参数列表、初始化器或聚合项：`"data types with sizes that aren't a power of two">,`。
- **L1032 EN**: Adds a standalone statement or declaration: `DefaultError, InGroup<IncompatibleMSStruct>;`.
  **L1032 CN**: 添加一条独立语句或声明：`DefaultError, InGroup<IncompatibleMSStruct>;`。

### Lines 1033-1056

````tablegen

def err_itanium_layout_unimplemented : Error<
  "Itanium-compatible layout for the Microsoft C++ ABI is not yet supported">;

// Unsupported features in name mangling
def err_unsupported_itanium_mangling : Error<
    "cannot yet mangle "
    "%enum_select<UnsupportedItaniumManglingKind>"
    "{%FixedPointLiteral{fixed point literal}"
    "|%DependentNeonVector{dependent neon vector type}"
    "|%DependentFixedLengthSVEVector{dependent fixed-length SVE vector "
    "type}"
    "|%DependentFixedLengthRVVVectorType{dependent fixed-length RVV "
    "vector type}"
    "|%TernaryWithOmittedMiddleOperand{?: operator with omitted middle "
    "operand}"
    "|%OpenACCAsteriskSizeExpr{OpenACC Asterisk Size expression}"
    "|%UnnamedUnionNTTP{unnamed union non-type template parameter}"
    "|%RequiresExprWithSubstitutionFailure{requires-expression "
    "containing a substitution failure}"
    "}0">;

def err_unsupported_itanium_expr_mangling : Error<
    "cannot yet mangle %0 expression">;
````
- **L1033 EN**: Blank line separating nearby declarations or logic blocks.
  **L1033 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1034 EN**: Declares TableGen def record `err_itanium_layout_unimplemented`.
  **L1034 CN**: 声明 TableGen def 记录 `err_itanium_layout_unimplemented`。
- **L1035 EN**: Adds a standalone statement or declaration: `"Itanium-compatible layout for the Microsoft C++ ABI is not yet supported">;`.
  **L1035 CN**: 添加一条独立语句或声明：`"Itanium-compatible layout for the Microsoft C++ ABI is not yet supported">;`。
- **L1036 EN**: Blank line separating nearby declarations or logic blocks.
  **L1036 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1037 EN**: Comment explains nearby logic, constraints, or intent: `Unsupported features in name mangling`.
  **L1037 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Unsupported features in name mangling`。
- **L1038 EN**: Declares TableGen def record `err_unsupported_itanium_mangling`.
  **L1038 CN**: 声明 TableGen def 记录 `err_unsupported_itanium_mangling`。
- **L1039 EN**: Continues the surrounding expression or declaration: `"cannot yet mangle "`.
  **L1039 CN**: 继续构造周围的表达式或声明：`"cannot yet mangle "`。
- **L1040 EN**: Continues the surrounding expression or declaration: `"%enum_select<UnsupportedItaniumManglingKind>"`.
  **L1040 CN**: 继续构造周围的表达式或声明：`"%enum_select<UnsupportedItaniumManglingKind>"`。
- **L1041 EN**: Continues the surrounding expression or declaration: `"{%FixedPointLiteral{fixed point literal}"`.
  **L1041 CN**: 继续构造周围的表达式或声明：`"{%FixedPointLiteral{fixed point literal}"`。
- **L1042 EN**: Continues the surrounding expression or declaration: `"|%DependentNeonVector{dependent neon vector type}"`.
  **L1042 CN**: 继续构造周围的表达式或声明：`"|%DependentNeonVector{dependent neon vector type}"`。
- **L1043 EN**: Continues the surrounding expression or declaration: `"|%DependentFixedLengthSVEVector{dependent fixed-length SVE vector "`.
  **L1043 CN**: 继续构造周围的表达式或声明：`"|%DependentFixedLengthSVEVector{dependent fixed-length SVE vector "`。
- **L1044 EN**: Continues the surrounding expression or declaration: `"type}"`.
  **L1044 CN**: 继续构造周围的表达式或声明：`"type}"`。
- **L1045 EN**: Continues the surrounding expression or declaration: `"|%DependentFixedLengthRVVVectorType{dependent fixed-length RVV "`.
  **L1045 CN**: 继续构造周围的表达式或声明：`"|%DependentFixedLengthRVVVectorType{dependent fixed-length RVV "`。
- **L1046 EN**: Continues the surrounding expression or declaration: `"vector type}"`.
  **L1046 CN**: 继续构造周围的表达式或声明：`"vector type}"`。
- **L1047 EN**: Continues the surrounding expression or declaration: `"|%TernaryWithOmittedMiddleOperand{?: operator with omitted middle "`.
  **L1047 CN**: 继续构造周围的表达式或声明：`"|%TernaryWithOmittedMiddleOperand{?: operator with omitted middle "`。
- **L1048 EN**: Continues the surrounding expression or declaration: `"operand}"`.
  **L1048 CN**: 继续构造周围的表达式或声明：`"operand}"`。
- **L1049 EN**: Continues the surrounding expression or declaration: `"|%OpenACCAsteriskSizeExpr{OpenACC Asterisk Size expression}"`.
  **L1049 CN**: 继续构造周围的表达式或声明：`"|%OpenACCAsteriskSizeExpr{OpenACC Asterisk Size expression}"`。
- **L1050 EN**: Continues the surrounding expression or declaration: `"|%UnnamedUnionNTTP{unnamed union non-type template parameter}"`.
  **L1050 CN**: 继续构造周围的表达式或声明：`"|%UnnamedUnionNTTP{unnamed union non-type template parameter}"`。
- **L1051 EN**: Continues the surrounding expression or declaration: `"|%RequiresExprWithSubstitutionFailure{requires-expression "`.
  **L1051 CN**: 继续构造周围的表达式或声明：`"|%RequiresExprWithSubstitutionFailure{requires-expression "`。
- **L1052 EN**: Continues the surrounding expression or declaration: `"containing a substitution failure}"`.
  **L1052 CN**: 继续构造周围的表达式或声明：`"containing a substitution failure}"`。
- **L1053 EN**: Adds a standalone statement or declaration: `"}0">;`.
  **L1053 CN**: 添加一条独立语句或声明：`"}0">;`。
- **L1054 EN**: Blank line separating nearby declarations or logic blocks.
  **L1054 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1055 EN**: Declares TableGen def record `err_unsupported_itanium_expr_mangling`.
  **L1055 CN**: 声明 TableGen def 记录 `err_unsupported_itanium_expr_mangling`。
- **L1056 EN**: Adds a standalone statement or declaration: `"cannot yet mangle %0 expression">;`.
  **L1056 CN**: 添加一条独立语句或声明：`"cannot yet mangle %0 expression">;`。

### Lines 1057-1080

````tablegen

def err_ms_mangle_number_overflow : Error<
    "mangling number exceeds limit (65535)">;
def err_ms_mangle_unsupported_with_detail : Error<
    "cannot mangle this %0 %1 yet">;
def err_ms_mangle_unsupported : Error<"cannot mangle this %0 yet">;

def err_unexpected_vftable_component : Error<
    "unexpected vftable component type %0 for component number %1">;

// -Wpadded-bitfield
def warn_padded_struct_bitfield : Warning<
  "padding %select{struct|interface|class}0 %1 with %2 "
  "%select{byte|bit}3%s2 to align %4">,
  InGroup<PaddedBitField>, DefaultIgnore;
def warn_padded_struct_anon_bitfield : Warning<
  "padding %select{struct|interface|class}0 %1 with %2 "
  "%select{byte|bit}3%s2 to align anonymous bit-field">,
  InGroup<PaddedBitField>, DefaultIgnore;

// -Wpadded, -Wpacked
def warn_padded_struct_field : Warning<
  "padding %select{struct|interface|class}0 %1 with %2 "
  "%select{byte|bit}3%s2 to align %4">,
````
- **L1057 EN**: Blank line separating nearby declarations or logic blocks.
  **L1057 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1058 EN**: Declares TableGen def record `err_ms_mangle_number_overflow`.
  **L1058 CN**: 声明 TableGen def 记录 `err_ms_mangle_number_overflow`。
- **L1059 EN**: Executes a call or declaration centered on `limit`.
  **L1059 CN**: 执行以 `limit` 为核心的调用或声明。
- **L1060 EN**: Declares TableGen def record `err_ms_mangle_unsupported_with_detail`.
  **L1060 CN**: 声明 TableGen def 记录 `err_ms_mangle_unsupported_with_detail`。
- **L1061 EN**: Adds a standalone statement or declaration: `"cannot mangle this %0 %1 yet">;`.
  **L1061 CN**: 添加一条独立语句或声明：`"cannot mangle this %0 %1 yet">;`。
- **L1062 EN**: Declares TableGen def record `err_ms_mangle_unsupported`.
  **L1062 CN**: 声明 TableGen def 记录 `err_ms_mangle_unsupported`。
- **L1063 EN**: Blank line separating nearby declarations or logic blocks.
  **L1063 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1064 EN**: Declares TableGen def record `err_unexpected_vftable_component`.
  **L1064 CN**: 声明 TableGen def 记录 `err_unexpected_vftable_component`。
- **L1065 EN**: Adds a standalone statement or declaration: `"unexpected vftable component type %0 for component number %1">;`.
  **L1065 CN**: 添加一条独立语句或声明：`"unexpected vftable component type %0 for component number %1">;`。
- **L1066 EN**: Blank line separating nearby declarations or logic blocks.
  **L1066 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1067 EN**: Comment explains nearby logic, constraints, or intent: `Wpadded-bitfield`.
  **L1067 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Wpadded-bitfield`。
- **L1068 EN**: Declares TableGen def record `warn_padded_struct_bitfield`.
  **L1068 CN**: 声明 TableGen def 记录 `warn_padded_struct_bitfield`。
- **L1069 EN**: Continues the surrounding expression or declaration: `"padding %select{struct|interface|class}0 %1 with %2 "`.
  **L1069 CN**: 继续构造周围的表达式或声明：`"padding %select{struct|interface|class}0 %1 with %2 "`。
- **L1070 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"%select{byte|bit}3%s2 to align %4">,`.
  **L1070 CN**: 继续一个多行参数列表、初始化器或聚合项：`"%select{byte|bit}3%s2 to align %4">,`。
- **L1071 EN**: Adds a standalone statement or declaration: `InGroup<PaddedBitField>, DefaultIgnore;`.
  **L1071 CN**: 添加一条独立语句或声明：`InGroup<PaddedBitField>, DefaultIgnore;`。
- **L1072 EN**: Declares TableGen def record `warn_padded_struct_anon_bitfield`.
  **L1072 CN**: 声明 TableGen def 记录 `warn_padded_struct_anon_bitfield`。
- **L1073 EN**: Continues the surrounding expression or declaration: `"padding %select{struct|interface|class}0 %1 with %2 "`.
  **L1073 CN**: 继续构造周围的表达式或声明：`"padding %select{struct|interface|class}0 %1 with %2 "`。
- **L1074 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"%select{byte|bit}3%s2 to align anonymous bit-field">,`.
  **L1074 CN**: 继续一个多行参数列表、初始化器或聚合项：`"%select{byte|bit}3%s2 to align anonymous bit-field">,`。
- **L1075 EN**: Adds a standalone statement or declaration: `InGroup<PaddedBitField>, DefaultIgnore;`.
  **L1075 CN**: 添加一条独立语句或声明：`InGroup<PaddedBitField>, DefaultIgnore;`。
- **L1076 EN**: Blank line separating nearby declarations or logic blocks.
  **L1076 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1077 EN**: Comment explains nearby logic, constraints, or intent: `Wpadded, -Wpacked`.
  **L1077 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Wpadded, -Wpacked`。
- **L1078 EN**: Declares TableGen def record `warn_padded_struct_field`.
  **L1078 CN**: 声明 TableGen def 记录 `warn_padded_struct_field`。
- **L1079 EN**: Continues the surrounding expression or declaration: `"padding %select{struct|interface|class}0 %1 with %2 "`.
  **L1079 CN**: 继续构造周围的表达式或声明：`"padding %select{struct|interface|class}0 %1 with %2 "`。
- **L1080 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"%select{byte|bit}3%s2 to align %4">,`.
  **L1080 CN**: 继续一个多行参数列表、初始化器或聚合项：`"%select{byte|bit}3%s2 to align %4">,`。

### Lines 1081-1101

````tablegen
  InGroup<Padded>, DefaultIgnore;
def warn_padded_struct_anon_field : Warning<
  "padding %select{struct|interface|class}0 %1 with %2 "
  "%select{byte|bit}3%s2 to align anonymous field">,
  InGroup<Padded>, DefaultIgnore;
def warn_padded_struct_size : Warning<
  "padding size of %0 with %1 %select{byte|bit}2%s1 to alignment boundary">,
  InGroup<Padded>, DefaultIgnore;
def warn_unnecessary_packed : Warning<
  "packed attribute is unnecessary for %0">, InGroup<Packed>, DefaultIgnore;
def warn_unpacked_field
    : Warning<
          "not packing field %0 as it is non-POD for the purposes of layout">,
      InGroup<PackedNonPod>,
      DefaultIgnore;

// -Wunaligned-access
def warn_unaligned_access : Warning<
  "field %1 within %0 is less aligned than %2 and is usually due to %0 being "
  "packed, which can lead to unaligned accesses">, InGroup<UnalignedAccess>, DefaultIgnore;
}
````
- **L1081 EN**: Adds a standalone statement or declaration: `InGroup<Padded>, DefaultIgnore;`.
  **L1081 CN**: 添加一条独立语句或声明：`InGroup<Padded>, DefaultIgnore;`。
- **L1082 EN**: Declares TableGen def record `warn_padded_struct_anon_field`.
  **L1082 CN**: 声明 TableGen def 记录 `warn_padded_struct_anon_field`。
- **L1083 EN**: Continues the surrounding expression or declaration: `"padding %select{struct|interface|class}0 %1 with %2 "`.
  **L1083 CN**: 继续构造周围的表达式或声明：`"padding %select{struct|interface|class}0 %1 with %2 "`。
- **L1084 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"%select{byte|bit}3%s2 to align anonymous field">,`.
  **L1084 CN**: 继续一个多行参数列表、初始化器或聚合项：`"%select{byte|bit}3%s2 to align anonymous field">,`。
- **L1085 EN**: Adds a standalone statement or declaration: `InGroup<Padded>, DefaultIgnore;`.
  **L1085 CN**: 添加一条独立语句或声明：`InGroup<Padded>, DefaultIgnore;`。
- **L1086 EN**: Declares TableGen def record `warn_padded_struct_size`.
  **L1086 CN**: 声明 TableGen def 记录 `warn_padded_struct_size`。
- **L1087 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"padding size of %0 with %1 %select{byte|bit}2%s1 to alignment boundary">,`.
  **L1087 CN**: 继续一个多行参数列表、初始化器或聚合项：`"padding size of %0 with %1 %select{byte|bit}2%s1 to alignment boundary">,`。
- **L1088 EN**: Adds a standalone statement or declaration: `InGroup<Padded>, DefaultIgnore;`.
  **L1088 CN**: 添加一条独立语句或声明：`InGroup<Padded>, DefaultIgnore;`。
- **L1089 EN**: Declares TableGen def record `warn_unnecessary_packed`.
  **L1089 CN**: 声明 TableGen def 记录 `warn_unnecessary_packed`。
- **L1090 EN**: Adds a standalone statement or declaration: `"packed attribute is unnecessary for %0">, InGroup<Packed>, DefaultIgnore;`.
  **L1090 CN**: 添加一条独立语句或声明：`"packed attribute is unnecessary for %0">, InGroup<Packed>, DefaultIgnore;`。
- **L1091 EN**: Declares TableGen def record `warn_unpacked_field`.
  **L1091 CN**: 声明 TableGen def 记录 `warn_unpacked_field`。
- **L1092 EN**: Continues the surrounding expression or declaration: `: Warning<`.
  **L1092 CN**: 继续构造周围的表达式或声明：`: Warning<`。
- **L1093 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"not packing field %0 as it is non-POD for the purposes of layout">,`.
  **L1093 CN**: 继续一个多行参数列表、初始化器或聚合项：`"not packing field %0 as it is non-POD for the purposes of layout">,`。
- **L1094 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `InGroup<PackedNonPod>,`.
  **L1094 CN**: 继续一个多行参数列表、初始化器或聚合项：`InGroup<PackedNonPod>,`。
- **L1095 EN**: Adds a standalone statement or declaration: `DefaultIgnore;`.
  **L1095 CN**: 添加一条独立语句或声明：`DefaultIgnore;`。
- **L1096 EN**: Blank line separating nearby declarations or logic blocks.
  **L1096 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1097 EN**: Comment explains nearby logic, constraints, or intent: `Wunaligned-access`.
  **L1097 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Wunaligned-access`。
- **L1098 EN**: Declares TableGen def record `warn_unaligned_access`.
  **L1098 CN**: 声明 TableGen def 记录 `warn_unaligned_access`。
- **L1099 EN**: Continues the surrounding expression or declaration: `"field %1 within %0 is less aligned than %2 and is usually due to %0 being "`.
  **L1099 CN**: 继续构造周围的表达式或声明：`"field %1 within %0 is less aligned than %2 and is usually due to %0 being "`。
- **L1100 EN**: Adds a standalone statement or declaration: `"packed, which can lead to unaligned accesses">, InGroup<UnalignedAccess>, DefaultIgnore;`.
  **L1100 CN**: 添加一条独立语句或声明：`"packed, which can lead to unaligned accesses">, InGroup<UnalignedAccess>, DefaultIgnore;`。
- **L1101 EN**: Closes the current lexical scope or compound statement.
  **L1101 CN**: 结束当前词法作用域或复合语句块。

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
- **Virtual dispatch layout / 虚派发布局**
  - **EN**: Encodes vtable structure, method slots, and base-adjustment information for C++.
  - **CN**: 编码 C++ 的虚表结构、方法槽位与基类调整信息。

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: None detected. / 未检测到。
- **Macros / 宏**: No prominent macros detected. / 未检测到明显宏。
- **Types / 类型**: `cannot`, `of`, `subobject`, `with`, `member`, `type`, `has`, `here`, `that`, `scoped`, `non`
- **Functions or callables / 函数或可调用对象**: `These`, `specifiers`, `object`, `pointer`, `copy`, `limit`, `units`, `parameters`
- **TableGen records / TableGen 记录**: `note_expr_divide_by_zero`, `note_constexpr_invalid_cast`, `note_constexpr_invalid_void_star_cast`, `note_constexpr_invalid_downcast`, `note_constexpr_overflow`, `note_constexpr_negative_shift`, `note_constexpr_large_shift`, `note_constexpr_lshift_of_negative`, `note_constexpr_lshift_discards`, `note_constexpr_invalid_function`, `note_constexpr_invalid_inhctor`, `note_constexpr_no_return`, `note_constexpr_virtual_call`, `note_constexpr_pure_virtual_call`, `note_constexpr_polymorphic_unknown_dynamic_type`, `note_constexpr_dynamic_cast_to_reference_failed`
- **Namespaces / 命名空间**: No explicit namespaces detected. / 未检测到显式命名空间。
