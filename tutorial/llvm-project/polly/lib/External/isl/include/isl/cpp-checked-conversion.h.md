# cpp-checked-conversion.h — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `polly/lib/External/isl/include/isl/cpp-checked-conversion.h` | `polly/lib/External/isl/include/isl/cpp-checked-conversion.h` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | Declares upstream ISL interfaces and C++ wrappers that Polly depends on for polyhedral operations. | 声明 Polly 依赖的上游 ISL 接口与 C++ 包装层，用于多面体计算。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````cpp
/// These are automatically generated conversions between
/// the default and the checked C++ bindings for isl.
///
/// isl is a library for computing with integer sets and maps described by
/// Presburger formulas. On top of this, isl provides various tools for
/// polyhedral compilation, ranging from dependence analysis over scheduling
/// to AST generation.

#ifndef ISL_CPP_CHECKED_CONVERSION
#define ISL_CPP_CHECKED_CONVERSION

#include <isl/cpp.h>
#include <isl/cpp-checked.h>

namespace isl {

checked::aff check(aff obj) {
	return checked::manage(obj.copy());
}

````
- **EN**: This block imports ISL headers needed by the surrounding code; uses preprocessor directives to control compilation, macros, or include guards; opens or organizes declarations inside a C++ namespace; defines macros like `ISL_CPP_CHECKED_CONVERSION`; and continues the surrounding implementation details.
- **CN**: 该代码块 引入周边逻辑所需的 ISL 头文件; 使用预处理指令控制编译、宏或头文件保护; 在 C++ 命名空间中组织声明或实现; 定义宏，例如 `ISL_CPP_CHECKED_CONVERSION`；并延续周边实现细节。

### Lines 21-40

````cpp
aff uncheck(checked::aff obj) {
	return manage(obj.copy());
}

checked::aff_list check(aff_list obj) {
	return checked::manage(obj.copy());
}

aff_list uncheck(checked::aff_list obj) {
	return manage(obj.copy());
}

checked::ast_build check(ast_build obj) {
	return checked::manage(obj.copy());
}

ast_build uncheck(checked::ast_build obj) {
	return manage(obj.copy());
}

````
- **EN**: This block declares or defines routines around `uncheck`, `check`; emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `uncheck`, `check` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 41-60

````cpp
checked::ast_expr check(ast_expr obj) {
	return checked::manage(obj.copy());
}

ast_expr uncheck(checked::ast_expr obj) {
	return manage(obj.copy());
}

checked::ast_expr_id check(ast_expr_id obj) {
	return checked::manage(obj.copy()).as<checked::ast_expr_id>();
}

ast_expr_id uncheck(checked::ast_expr_id obj) {
	return manage(obj.copy()).as<ast_expr_id>();
}

checked::ast_expr_int check(ast_expr_int obj) {
	return checked::manage(obj.copy()).as<checked::ast_expr_int>();
}

````
- **EN**: This block declares or defines routines around `check`, `uncheck`; emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `check`, `uncheck` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 61-80

````cpp
ast_expr_int uncheck(checked::ast_expr_int obj) {
	return manage(obj.copy()).as<ast_expr_int>();
}

checked::ast_expr_op check(ast_expr_op obj) {
	return checked::manage(obj.copy()).as<checked::ast_expr_op>();
}

ast_expr_op uncheck(checked::ast_expr_op obj) {
	return manage(obj.copy()).as<ast_expr_op>();
}

checked::ast_expr_op_access check(ast_expr_op_access obj) {
	return checked::manage(obj.copy()).as<checked::ast_expr_op>().as<checked::ast_expr_op_access>();
}

ast_expr_op_access uncheck(checked::ast_expr_op_access obj) {
	return manage(obj.copy()).as<ast_expr_op>().as<ast_expr_op_access>();
}

````
- **EN**: This block declares or defines routines around `uncheck`, `check`; emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `uncheck`, `check` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 81-100

````cpp
checked::ast_expr_op_add check(ast_expr_op_add obj) {
	return checked::manage(obj.copy()).as<checked::ast_expr_op>().as<checked::ast_expr_op_add>();
}

ast_expr_op_add uncheck(checked::ast_expr_op_add obj) {
	return manage(obj.copy()).as<ast_expr_op>().as<ast_expr_op_add>();
}

checked::ast_expr_op_address_of check(ast_expr_op_address_of obj) {
	return checked::manage(obj.copy()).as<checked::ast_expr_op>().as<checked::ast_expr_op_address_of>();
}

ast_expr_op_address_of uncheck(checked::ast_expr_op_address_of obj) {
	return manage(obj.copy()).as<ast_expr_op>().as<ast_expr_op_address_of>();
}

checked::ast_expr_op_and check(ast_expr_op_and obj) {
	return checked::manage(obj.copy()).as<checked::ast_expr_op>().as<checked::ast_expr_op_and>();
}

````
- **EN**: This block declares or defines routines around `check`, `uncheck`; emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `check`, `uncheck` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 101-120

````cpp
ast_expr_op_and uncheck(checked::ast_expr_op_and obj) {
	return manage(obj.copy()).as<ast_expr_op>().as<ast_expr_op_and>();
}

checked::ast_expr_op_and_then check(ast_expr_op_and_then obj) {
	return checked::manage(obj.copy()).as<checked::ast_expr_op>().as<checked::ast_expr_op_and_then>();
}

ast_expr_op_and_then uncheck(checked::ast_expr_op_and_then obj) {
	return manage(obj.copy()).as<ast_expr_op>().as<ast_expr_op_and_then>();
}

checked::ast_expr_op_call check(ast_expr_op_call obj) {
	return checked::manage(obj.copy()).as<checked::ast_expr_op>().as<checked::ast_expr_op_call>();
}

ast_expr_op_call uncheck(checked::ast_expr_op_call obj) {
	return manage(obj.copy()).as<ast_expr_op>().as<ast_expr_op_call>();
}

````
- **EN**: This block declares or defines routines around `uncheck`, `check`; emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `uncheck`, `check` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 121-140

````cpp
checked::ast_expr_op_cond check(ast_expr_op_cond obj) {
	return checked::manage(obj.copy()).as<checked::ast_expr_op>().as<checked::ast_expr_op_cond>();
}

ast_expr_op_cond uncheck(checked::ast_expr_op_cond obj) {
	return manage(obj.copy()).as<ast_expr_op>().as<ast_expr_op_cond>();
}

checked::ast_expr_op_div check(ast_expr_op_div obj) {
	return checked::manage(obj.copy()).as<checked::ast_expr_op>().as<checked::ast_expr_op_div>();
}

ast_expr_op_div uncheck(checked::ast_expr_op_div obj) {
	return manage(obj.copy()).as<ast_expr_op>().as<ast_expr_op_div>();
}

checked::ast_expr_op_eq check(ast_expr_op_eq obj) {
	return checked::manage(obj.copy()).as<checked::ast_expr_op>().as<checked::ast_expr_op_eq>();
}

````
- **EN**: This block declares or defines routines around `check`, `uncheck`; emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `check`, `uncheck` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 141-160

````cpp
ast_expr_op_eq uncheck(checked::ast_expr_op_eq obj) {
	return manage(obj.copy()).as<ast_expr_op>().as<ast_expr_op_eq>();
}

checked::ast_expr_op_fdiv_q check(ast_expr_op_fdiv_q obj) {
	return checked::manage(obj.copy()).as<checked::ast_expr_op>().as<checked::ast_expr_op_fdiv_q>();
}

ast_expr_op_fdiv_q uncheck(checked::ast_expr_op_fdiv_q obj) {
	return manage(obj.copy()).as<ast_expr_op>().as<ast_expr_op_fdiv_q>();
}

checked::ast_expr_op_ge check(ast_expr_op_ge obj) {
	return checked::manage(obj.copy()).as<checked::ast_expr_op>().as<checked::ast_expr_op_ge>();
}

ast_expr_op_ge uncheck(checked::ast_expr_op_ge obj) {
	return manage(obj.copy()).as<ast_expr_op>().as<ast_expr_op_ge>();
}

````
- **EN**: This block declares or defines routines around `uncheck`, `check`; emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `uncheck`, `check` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 161-180

````cpp
checked::ast_expr_op_gt check(ast_expr_op_gt obj) {
	return checked::manage(obj.copy()).as<checked::ast_expr_op>().as<checked::ast_expr_op_gt>();
}

ast_expr_op_gt uncheck(checked::ast_expr_op_gt obj) {
	return manage(obj.copy()).as<ast_expr_op>().as<ast_expr_op_gt>();
}

checked::ast_expr_op_le check(ast_expr_op_le obj) {
	return checked::manage(obj.copy()).as<checked::ast_expr_op>().as<checked::ast_expr_op_le>();
}

ast_expr_op_le uncheck(checked::ast_expr_op_le obj) {
	return manage(obj.copy()).as<ast_expr_op>().as<ast_expr_op_le>();
}

checked::ast_expr_op_lt check(ast_expr_op_lt obj) {
	return checked::manage(obj.copy()).as<checked::ast_expr_op>().as<checked::ast_expr_op_lt>();
}

````
- **EN**: This block declares or defines routines around `check`, `uncheck`; emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `check`, `uncheck` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 181-200

````cpp
ast_expr_op_lt uncheck(checked::ast_expr_op_lt obj) {
	return manage(obj.copy()).as<ast_expr_op>().as<ast_expr_op_lt>();
}

checked::ast_expr_op_max check(ast_expr_op_max obj) {
	return checked::manage(obj.copy()).as<checked::ast_expr_op>().as<checked::ast_expr_op_max>();
}

ast_expr_op_max uncheck(checked::ast_expr_op_max obj) {
	return manage(obj.copy()).as<ast_expr_op>().as<ast_expr_op_max>();
}

checked::ast_expr_op_member check(ast_expr_op_member obj) {
	return checked::manage(obj.copy()).as<checked::ast_expr_op>().as<checked::ast_expr_op_member>();
}

ast_expr_op_member uncheck(checked::ast_expr_op_member obj) {
	return manage(obj.copy()).as<ast_expr_op>().as<ast_expr_op_member>();
}

````
- **EN**: This block declares or defines routines around `uncheck`, `check`; emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `uncheck`, `check` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 201-220

````cpp
checked::ast_expr_op_min check(ast_expr_op_min obj) {
	return checked::manage(obj.copy()).as<checked::ast_expr_op>().as<checked::ast_expr_op_min>();
}

ast_expr_op_min uncheck(checked::ast_expr_op_min obj) {
	return manage(obj.copy()).as<ast_expr_op>().as<ast_expr_op_min>();
}

checked::ast_expr_op_minus check(ast_expr_op_minus obj) {
	return checked::manage(obj.copy()).as<checked::ast_expr_op>().as<checked::ast_expr_op_minus>();
}

ast_expr_op_minus uncheck(checked::ast_expr_op_minus obj) {
	return manage(obj.copy()).as<ast_expr_op>().as<ast_expr_op_minus>();
}

checked::ast_expr_op_mul check(ast_expr_op_mul obj) {
	return checked::manage(obj.copy()).as<checked::ast_expr_op>().as<checked::ast_expr_op_mul>();
}

````
- **EN**: This block declares or defines routines around `check`, `uncheck`; emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `check`, `uncheck` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 221-240

````cpp
ast_expr_op_mul uncheck(checked::ast_expr_op_mul obj) {
	return manage(obj.copy()).as<ast_expr_op>().as<ast_expr_op_mul>();
}

checked::ast_expr_op_or check(ast_expr_op_or obj) {
	return checked::manage(obj.copy()).as<checked::ast_expr_op>().as<checked::ast_expr_op_or>();
}

ast_expr_op_or uncheck(checked::ast_expr_op_or obj) {
	return manage(obj.copy()).as<ast_expr_op>().as<ast_expr_op_or>();
}

checked::ast_expr_op_or_else check(ast_expr_op_or_else obj) {
	return checked::manage(obj.copy()).as<checked::ast_expr_op>().as<checked::ast_expr_op_or_else>();
}

ast_expr_op_or_else uncheck(checked::ast_expr_op_or_else obj) {
	return manage(obj.copy()).as<ast_expr_op>().as<ast_expr_op_or_else>();
}

````
- **EN**: This block declares or defines routines around `uncheck`, `check`; emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `uncheck`, `check` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 241-260

````cpp
checked::ast_expr_op_pdiv_q check(ast_expr_op_pdiv_q obj) {
	return checked::manage(obj.copy()).as<checked::ast_expr_op>().as<checked::ast_expr_op_pdiv_q>();
}

ast_expr_op_pdiv_q uncheck(checked::ast_expr_op_pdiv_q obj) {
	return manage(obj.copy()).as<ast_expr_op>().as<ast_expr_op_pdiv_q>();
}

checked::ast_expr_op_pdiv_r check(ast_expr_op_pdiv_r obj) {
	return checked::manage(obj.copy()).as<checked::ast_expr_op>().as<checked::ast_expr_op_pdiv_r>();
}

ast_expr_op_pdiv_r uncheck(checked::ast_expr_op_pdiv_r obj) {
	return manage(obj.copy()).as<ast_expr_op>().as<ast_expr_op_pdiv_r>();
}

checked::ast_expr_op_select check(ast_expr_op_select obj) {
	return checked::manage(obj.copy()).as<checked::ast_expr_op>().as<checked::ast_expr_op_select>();
}

````
- **EN**: This block declares or defines routines around `check`, `uncheck`; emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `check`, `uncheck` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 261-280

````cpp
ast_expr_op_select uncheck(checked::ast_expr_op_select obj) {
	return manage(obj.copy()).as<ast_expr_op>().as<ast_expr_op_select>();
}

checked::ast_expr_op_sub check(ast_expr_op_sub obj) {
	return checked::manage(obj.copy()).as<checked::ast_expr_op>().as<checked::ast_expr_op_sub>();
}

ast_expr_op_sub uncheck(checked::ast_expr_op_sub obj) {
	return manage(obj.copy()).as<ast_expr_op>().as<ast_expr_op_sub>();
}

checked::ast_expr_op_zdiv_r check(ast_expr_op_zdiv_r obj) {
	return checked::manage(obj.copy()).as<checked::ast_expr_op>().as<checked::ast_expr_op_zdiv_r>();
}

ast_expr_op_zdiv_r uncheck(checked::ast_expr_op_zdiv_r obj) {
	return manage(obj.copy()).as<ast_expr_op>().as<ast_expr_op_zdiv_r>();
}

````
- **EN**: This block declares or defines routines around `uncheck`, `check`; emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `uncheck`, `check` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 281-300

````cpp
checked::ast_node check(ast_node obj) {
	return checked::manage(obj.copy());
}

ast_node uncheck(checked::ast_node obj) {
	return manage(obj.copy());
}

checked::ast_node_block check(ast_node_block obj) {
	return checked::manage(obj.copy()).as<checked::ast_node_block>();
}

ast_node_block uncheck(checked::ast_node_block obj) {
	return manage(obj.copy()).as<ast_node_block>();
}

checked::ast_node_for check(ast_node_for obj) {
	return checked::manage(obj.copy()).as<checked::ast_node_for>();
}

````
- **EN**: This block declares or defines routines around `check`, `uncheck`; emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `check`, `uncheck` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 301-320

````cpp
ast_node_for uncheck(checked::ast_node_for obj) {
	return manage(obj.copy()).as<ast_node_for>();
}

checked::ast_node_if check(ast_node_if obj) {
	return checked::manage(obj.copy()).as<checked::ast_node_if>();
}

ast_node_if uncheck(checked::ast_node_if obj) {
	return manage(obj.copy()).as<ast_node_if>();
}

checked::ast_node_list check(ast_node_list obj) {
	return checked::manage(obj.copy());
}

ast_node_list uncheck(checked::ast_node_list obj) {
	return manage(obj.copy());
}

````
- **EN**: This block declares or defines routines around `uncheck`, `check`; emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `uncheck`, `check` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 321-340

````cpp
checked::ast_node_mark check(ast_node_mark obj) {
	return checked::manage(obj.copy()).as<checked::ast_node_mark>();
}

ast_node_mark uncheck(checked::ast_node_mark obj) {
	return manage(obj.copy()).as<ast_node_mark>();
}

checked::ast_node_user check(ast_node_user obj) {
	return checked::manage(obj.copy()).as<checked::ast_node_user>();
}

ast_node_user uncheck(checked::ast_node_user obj) {
	return manage(obj.copy()).as<ast_node_user>();
}

checked::basic_map check(basic_map obj) {
	return checked::manage(obj.copy());
}

````
- **EN**: This block declares or defines routines around `check`, `uncheck`; emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `check`, `uncheck` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 341-360

````cpp
basic_map uncheck(checked::basic_map obj) {
	return manage(obj.copy());
}

checked::basic_set check(basic_set obj) {
	return checked::manage(obj.copy());
}

basic_set uncheck(checked::basic_set obj) {
	return manage(obj.copy());
}

checked::fixed_box check(fixed_box obj) {
	return checked::manage(obj.copy());
}

fixed_box uncheck(checked::fixed_box obj) {
	return manage(obj.copy());
}

````
- **EN**: This block declares or defines routines around `uncheck`, `check`; emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `uncheck`, `check` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 361-380

````cpp
checked::id check(id obj) {
	return checked::manage(obj.copy());
}

id uncheck(checked::id obj) {
	return manage(obj.copy());
}

checked::id_list check(id_list obj) {
	return checked::manage(obj.copy());
}

id_list uncheck(checked::id_list obj) {
	return manage(obj.copy());
}

checked::id_to_ast_expr check(id_to_ast_expr obj) {
	return checked::manage(obj.copy());
}

````
- **EN**: This block declares or defines routines around `check`, `uncheck`; emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `check`, `uncheck` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 381-400

````cpp
id_to_ast_expr uncheck(checked::id_to_ast_expr obj) {
	return manage(obj.copy());
}

checked::id_to_id check(id_to_id obj) {
	return checked::manage(obj.copy());
}

id_to_id uncheck(checked::id_to_id obj) {
	return manage(obj.copy());
}

checked::map check(map obj) {
	return checked::manage(obj.copy());
}

map uncheck(checked::map obj) {
	return manage(obj.copy());
}

````
- **EN**: This block declares or defines routines around `uncheck`, `check`; emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `uncheck`, `check` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 401-420

````cpp
checked::map_list check(map_list obj) {
	return checked::manage(obj.copy());
}

map_list uncheck(checked::map_list obj) {
	return manage(obj.copy());
}

checked::multi_aff check(multi_aff obj) {
	return checked::manage(obj.copy());
}

multi_aff uncheck(checked::multi_aff obj) {
	return manage(obj.copy());
}

checked::multi_id check(multi_id obj) {
	return checked::manage(obj.copy());
}

````
- **EN**: This block declares or defines routines around `check`, `uncheck`; emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `check`, `uncheck` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 421-440

````cpp
multi_id uncheck(checked::multi_id obj) {
	return manage(obj.copy());
}

checked::multi_pw_aff check(multi_pw_aff obj) {
	return checked::manage(obj.copy());
}

multi_pw_aff uncheck(checked::multi_pw_aff obj) {
	return manage(obj.copy());
}

checked::multi_union_pw_aff check(multi_union_pw_aff obj) {
	return checked::manage(obj.copy());
}

multi_union_pw_aff uncheck(checked::multi_union_pw_aff obj) {
	return manage(obj.copy());
}

````
- **EN**: This block declares or defines routines around `uncheck`, `check`; emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `uncheck`, `check` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 441-460

````cpp
checked::multi_val check(multi_val obj) {
	return checked::manage(obj.copy());
}

multi_val uncheck(checked::multi_val obj) {
	return manage(obj.copy());
}

checked::point check(point obj) {
	return checked::manage(obj.copy());
}

point uncheck(checked::point obj) {
	return manage(obj.copy());
}

checked::pw_aff check(pw_aff obj) {
	return checked::manage(obj.copy());
}

````
- **EN**: This block declares or defines routines around `check`, `uncheck`; emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `check`, `uncheck` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 461-480

````cpp
pw_aff uncheck(checked::pw_aff obj) {
	return manage(obj.copy());
}

checked::pw_aff_list check(pw_aff_list obj) {
	return checked::manage(obj.copy());
}

pw_aff_list uncheck(checked::pw_aff_list obj) {
	return manage(obj.copy());
}

checked::pw_multi_aff check(pw_multi_aff obj) {
	return checked::manage(obj.copy());
}

pw_multi_aff uncheck(checked::pw_multi_aff obj) {
	return manage(obj.copy());
}

````
- **EN**: This block declares or defines routines around `uncheck`, `check`; emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `uncheck`, `check` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 481-500

````cpp
checked::pw_multi_aff_list check(pw_multi_aff_list obj) {
	return checked::manage(obj.copy());
}

pw_multi_aff_list uncheck(checked::pw_multi_aff_list obj) {
	return manage(obj.copy());
}

checked::schedule check(schedule obj) {
	return checked::manage(obj.copy());
}

schedule uncheck(checked::schedule obj) {
	return manage(obj.copy());
}

checked::schedule_constraints check(schedule_constraints obj) {
	return checked::manage(obj.copy());
}

````
- **EN**: This block declares or defines routines around `check`, `uncheck`; emits return paths that hand results or status codes back to callers; mentions schedule-related state that orders statements or iterations.
- **CN**: 该代码块 声明或定义与 `check`, `uncheck` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 涉及调度相关状态，用于安排语句或迭代顺序.

### Lines 501-520

````cpp
schedule_constraints uncheck(checked::schedule_constraints obj) {
	return manage(obj.copy());
}

checked::schedule_node check(schedule_node obj) {
	return checked::manage(obj.copy());
}

schedule_node uncheck(checked::schedule_node obj) {
	return manage(obj.copy());
}

checked::schedule_node_band check(schedule_node_band obj) {
	return checked::manage(obj.copy()).as<checked::schedule_node_band>();
}

schedule_node_band uncheck(checked::schedule_node_band obj) {
	return manage(obj.copy()).as<schedule_node_band>();
}

````
- **EN**: This block declares or defines routines around `uncheck`, `check`; emits return paths that hand results or status codes back to callers; mentions schedule-related state that orders statements or iterations.
- **CN**: 该代码块 声明或定义与 `uncheck`, `check` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 涉及调度相关状态，用于安排语句或迭代顺序.

### Lines 521-540

````cpp
checked::schedule_node_context check(schedule_node_context obj) {
	return checked::manage(obj.copy()).as<checked::schedule_node_context>();
}

schedule_node_context uncheck(checked::schedule_node_context obj) {
	return manage(obj.copy()).as<schedule_node_context>();
}

checked::schedule_node_domain check(schedule_node_domain obj) {
	return checked::manage(obj.copy()).as<checked::schedule_node_domain>();
}

schedule_node_domain uncheck(checked::schedule_node_domain obj) {
	return manage(obj.copy()).as<schedule_node_domain>();
}

checked::schedule_node_expansion check(schedule_node_expansion obj) {
	return checked::manage(obj.copy()).as<checked::schedule_node_expansion>();
}

````
- **EN**: This block declares or defines routines around `check`, `uncheck`; emits return paths that hand results or status codes back to callers; mentions schedule-related state that orders statements or iterations.
- **CN**: 该代码块 声明或定义与 `check`, `uncheck` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 涉及调度相关状态，用于安排语句或迭代顺序.

### Lines 541-560

````cpp
schedule_node_expansion uncheck(checked::schedule_node_expansion obj) {
	return manage(obj.copy()).as<schedule_node_expansion>();
}

checked::schedule_node_extension check(schedule_node_extension obj) {
	return checked::manage(obj.copy()).as<checked::schedule_node_extension>();
}

schedule_node_extension uncheck(checked::schedule_node_extension obj) {
	return manage(obj.copy()).as<schedule_node_extension>();
}

checked::schedule_node_filter check(schedule_node_filter obj) {
	return checked::manage(obj.copy()).as<checked::schedule_node_filter>();
}

schedule_node_filter uncheck(checked::schedule_node_filter obj) {
	return manage(obj.copy()).as<schedule_node_filter>();
}

````
- **EN**: This block declares or defines routines around `uncheck`, `check`; emits return paths that hand results or status codes back to callers; mentions schedule-related state that orders statements or iterations.
- **CN**: 该代码块 声明或定义与 `uncheck`, `check` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 涉及调度相关状态，用于安排语句或迭代顺序.

### Lines 561-580

````cpp
checked::schedule_node_guard check(schedule_node_guard obj) {
	return checked::manage(obj.copy()).as<checked::schedule_node_guard>();
}

schedule_node_guard uncheck(checked::schedule_node_guard obj) {
	return manage(obj.copy()).as<schedule_node_guard>();
}

checked::schedule_node_leaf check(schedule_node_leaf obj) {
	return checked::manage(obj.copy()).as<checked::schedule_node_leaf>();
}

schedule_node_leaf uncheck(checked::schedule_node_leaf obj) {
	return manage(obj.copy()).as<schedule_node_leaf>();
}

checked::schedule_node_mark check(schedule_node_mark obj) {
	return checked::manage(obj.copy()).as<checked::schedule_node_mark>();
}

````
- **EN**: This block declares or defines routines around `check`, `uncheck`; emits return paths that hand results or status codes back to callers; mentions schedule-related state that orders statements or iterations.
- **CN**: 该代码块 声明或定义与 `check`, `uncheck` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 涉及调度相关状态，用于安排语句或迭代顺序.

### Lines 581-600

````cpp
schedule_node_mark uncheck(checked::schedule_node_mark obj) {
	return manage(obj.copy()).as<schedule_node_mark>();
}

checked::schedule_node_sequence check(schedule_node_sequence obj) {
	return checked::manage(obj.copy()).as<checked::schedule_node_sequence>();
}

schedule_node_sequence uncheck(checked::schedule_node_sequence obj) {
	return manage(obj.copy()).as<schedule_node_sequence>();
}

checked::schedule_node_set check(schedule_node_set obj) {
	return checked::manage(obj.copy()).as<checked::schedule_node_set>();
}

schedule_node_set uncheck(checked::schedule_node_set obj) {
	return manage(obj.copy()).as<schedule_node_set>();
}

````
- **EN**: This block declares or defines routines around `uncheck`, `check`; emits return paths that hand results or status codes back to callers; mentions schedule-related state that orders statements or iterations.
- **CN**: 该代码块 声明或定义与 `uncheck`, `check` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 涉及调度相关状态，用于安排语句或迭代顺序.

### Lines 601-620

````cpp
checked::set check(set obj) {
	return checked::manage(obj.copy());
}

set uncheck(checked::set obj) {
	return manage(obj.copy());
}

checked::set_list check(set_list obj) {
	return checked::manage(obj.copy());
}

set_list uncheck(checked::set_list obj) {
	return manage(obj.copy());
}

checked::space check(space obj) {
	return checked::manage(obj.copy());
}

````
- **EN**: This block declares or defines routines around `check`, `uncheck`; emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `check`, `uncheck` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 621-640

````cpp
space uncheck(checked::space obj) {
	return manage(obj.copy());
}

checked::union_access_info check(union_access_info obj) {
	return checked::manage(obj.copy());
}

union_access_info uncheck(checked::union_access_info obj) {
	return manage(obj.copy());
}

checked::union_flow check(union_flow obj) {
	return checked::manage(obj.copy());
}

union_flow uncheck(checked::union_flow obj) {
	return manage(obj.copy());
}

````
- **EN**: This block declares or defines routines around `uncheck`, `check`; emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `uncheck`, `check` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 641-660

````cpp
checked::union_map check(union_map obj) {
	return checked::manage(obj.copy());
}

union_map uncheck(checked::union_map obj) {
	return manage(obj.copy());
}

checked::union_pw_aff check(union_pw_aff obj) {
	return checked::manage(obj.copy());
}

union_pw_aff uncheck(checked::union_pw_aff obj) {
	return manage(obj.copy());
}

checked::union_pw_aff_list check(union_pw_aff_list obj) {
	return checked::manage(obj.copy());
}

````
- **EN**: This block declares or defines routines around `check`, `uncheck`; emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `check`, `uncheck` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 661-680

````cpp
union_pw_aff_list uncheck(checked::union_pw_aff_list obj) {
	return manage(obj.copy());
}

checked::union_pw_multi_aff check(union_pw_multi_aff obj) {
	return checked::manage(obj.copy());
}

union_pw_multi_aff uncheck(checked::union_pw_multi_aff obj) {
	return manage(obj.copy());
}

checked::union_set check(union_set obj) {
	return checked::manage(obj.copy());
}

union_set uncheck(checked::union_set obj) {
	return manage(obj.copy());
}

````
- **EN**: This block declares or defines routines around `uncheck`, `check`; emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `uncheck`, `check` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 681-700

````cpp
checked::union_set_list check(union_set_list obj) {
	return checked::manage(obj.copy());
}

union_set_list uncheck(checked::union_set_list obj) {
	return manage(obj.copy());
}

checked::val check(val obj) {
	return checked::manage(obj.copy());
}

val uncheck(checked::val obj) {
	return manage(obj.copy());
}

checked::val_list check(val_list obj) {
	return checked::manage(obj.copy());
}

````
- **EN**: This block declares or defines routines around `check`, `uncheck`; emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `check`, `uncheck` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 701-707

````cpp
val_list uncheck(checked::val_list obj) {
	return manage(obj.copy());
}

} // namespace isl

#endif /* ISL_CPP_CHECKED_CONVERSION */
````
- **EN**: This block uses preprocessor directives to control compilation, macros, or include guards; declares or defines routines around `uncheck`; emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 使用预处理指令控制编译、宏或头文件保护; 声明或定义与 `uncheck` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码.

## Key Concepts / 关键概念

- **Schedule construction**
  - **CN**: 调度构建
- **ISL relations and sets**
  - **CN**: ISL 关系与集合
- **Dependence analysis**
  - **CN**: 依赖分析
- **Public interface declarations**
  - **CN**: 公共接口声明

## Dependencies / 依赖关系

- **ISL headers**: `isl/cpp.h`, `isl/cpp-checked.h` — ISL relation/set primitives that support polyhedral analysis and transformations.
  **ISL headers（CN）**：`isl/cpp.h`, `isl/cpp-checked.h` —— 支撑多面体分析与变换的 ISL 关系/集合原语。
