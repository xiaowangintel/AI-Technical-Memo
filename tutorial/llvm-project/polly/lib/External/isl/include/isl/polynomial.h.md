# polynomial.h — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `polly/lib/External/isl/include/isl/polynomial.h` | `polly/lib/External/isl/include/isl/polynomial.h` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | Declares upstream ISL interfaces and C++ wrappers that Polly depends on for polyhedral operations. | 声明 Polly 依赖的上游 ISL 接口与 C++ 包装层，用于多面体计算。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-27

````cpp
#ifndef ISL_POLYNOMIAL_H
#define ISL_POLYNOMIAL_H

#include <isl/ctx.h>
#include <isl/constraint.h>
#include <isl/space_type.h>
#include <isl/set_type.h>
#include <isl/point.h>
#include <isl/printer.h>
#include <isl/union_set_type.h>
#include <isl/aff_type.h>
#include <isl/polynomial_type.h>
#include <isl/val_type.h>

#if defined(__cplusplus)
extern "C" {
#endif

isl_ctx *isl_qpolynomial_get_ctx(__isl_keep isl_qpolynomial *qp);
__isl_give isl_space *isl_qpolynomial_get_domain_space(
	__isl_keep isl_qpolynomial *qp);
__isl_give isl_space *isl_qpolynomial_get_space(__isl_keep isl_qpolynomial *qp);
isl_size isl_qpolynomial_dim(__isl_keep isl_qpolynomial *qp,
	enum isl_dim_type type);
isl_bool isl_qpolynomial_involves_dims(__isl_keep isl_qpolynomial *qp,
	enum isl_dim_type type, unsigned first, unsigned n);

````
- **EN**: This block imports ISL headers needed by the surrounding code; uses preprocessor directives to control compilation, macros, or include guards; defines enum values such as `isl_dim_type`; defines macros like `ISL_POLYNOMIAL_H`; and continues the surrounding implementation details.
- **CN**: 该代码块 引入周边逻辑所需的 ISL 头文件; 使用预处理指令控制编译、宏或头文件保护; 定义枚举类型，例如 `isl_dim_type`; 定义宏，例如 `ISL_POLYNOMIAL_H`；并延续周边实现细节。

### Lines 28-53

````cpp
__isl_give isl_val *isl_qpolynomial_get_constant_val(
	__isl_keep isl_qpolynomial *qp);

__isl_give isl_qpolynomial *isl_qpolynomial_set_dim_name(
	__isl_take isl_qpolynomial *qp,
	enum isl_dim_type type, unsigned pos, const char *s);

__isl_give isl_qpolynomial *isl_qpolynomial_zero_on_domain(
	__isl_take isl_space *domain);
__isl_give isl_qpolynomial *isl_qpolynomial_one_on_domain(
	__isl_take isl_space *domain);
__isl_give isl_qpolynomial *isl_qpolynomial_infty_on_domain(
	__isl_take isl_space *domain);
__isl_give isl_qpolynomial *isl_qpolynomial_neginfty_on_domain(
	__isl_take isl_space *domain);
__isl_give isl_qpolynomial *isl_qpolynomial_nan_on_domain(
	__isl_take isl_space *domain);
__isl_give isl_qpolynomial *isl_qpolynomial_val_on_domain(
	__isl_take isl_space *space, __isl_take isl_val *val);
__isl_give isl_qpolynomial *isl_qpolynomial_var_on_domain(
	__isl_take isl_space *domain,
	enum isl_dim_type type, unsigned pos);
__isl_give isl_qpolynomial *isl_qpolynomial_copy(__isl_keep isl_qpolynomial *qp);
__isl_null isl_qpolynomial *isl_qpolynomial_free(
	__isl_take isl_qpolynomial *qp);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_qpolynomial_get_constant_val`, `isl_qpolynomial_set_dim_name`, `isl_qpolynomial_zero_on_domain`, `isl_qpolynomial_one_on_domain` (+7 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_qpolynomial_get_constant_val`, `isl_qpolynomial_set_dim_name`, `isl_qpolynomial_zero_on_domain`, `isl_qpolynomial_one_on_domain` (+7 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 54-75

````cpp
isl_bool isl_qpolynomial_plain_is_equal(__isl_keep isl_qpolynomial *qp1,
	__isl_keep isl_qpolynomial *qp2);
isl_bool isl_qpolynomial_is_zero(__isl_keep isl_qpolynomial *qp);
isl_bool isl_qpolynomial_is_nan(__isl_keep isl_qpolynomial *qp);
isl_bool isl_qpolynomial_is_infty(__isl_keep isl_qpolynomial *qp);
isl_bool isl_qpolynomial_is_neginfty(__isl_keep isl_qpolynomial *qp);
int isl_qpolynomial_sgn(__isl_keep isl_qpolynomial *qp);

__isl_give isl_qpolynomial *isl_qpolynomial_neg(__isl_take isl_qpolynomial *qp);
__isl_give isl_qpolynomial *isl_qpolynomial_add(__isl_take isl_qpolynomial *qp1,
	__isl_take isl_qpolynomial *qp2);
__isl_give isl_qpolynomial *isl_qpolynomial_sub(__isl_take isl_qpolynomial *qp1,
	__isl_take isl_qpolynomial *qp2);
__isl_give isl_qpolynomial *isl_qpolynomial_mul(__isl_take isl_qpolynomial *qp1,
	__isl_take isl_qpolynomial *qp2);
__isl_give isl_qpolynomial *isl_qpolynomial_pow(__isl_take isl_qpolynomial *qp,
	unsigned power);
__isl_give isl_qpolynomial *isl_qpolynomial_scale_val(
	__isl_take isl_qpolynomial *qp, __isl_take isl_val *v);
__isl_give isl_qpolynomial *isl_qpolynomial_scale_down_val(
	__isl_take isl_qpolynomial *qp, __isl_take isl_val *v);

````
- **EN**: This block declares or defines routines around `isl_qpolynomial_plain_is_equal`, `isl_qpolynomial_is_zero`, `isl_qpolynomial_is_nan`, `isl_qpolynomial_is_infty` (+9 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_qpolynomial_plain_is_equal`, `isl_qpolynomial_is_zero`, `isl_qpolynomial_is_nan`, `isl_qpolynomial_is_infty` (+9 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 76-97

````cpp
__isl_give isl_qpolynomial *isl_qpolynomial_domain_reverse(
	__isl_take isl_qpolynomial *qp);
__isl_give isl_qpolynomial *isl_qpolynomial_insert_dims(
	__isl_take isl_qpolynomial *qp, enum isl_dim_type type,
	unsigned first, unsigned n);
__isl_give isl_qpolynomial *isl_qpolynomial_add_dims(
	__isl_take isl_qpolynomial *qp, enum isl_dim_type type, unsigned n);
__isl_give isl_qpolynomial *isl_qpolynomial_move_dims(
	__isl_take isl_qpolynomial *qp,
	enum isl_dim_type dst_type, unsigned dst_pos,
	enum isl_dim_type src_type, unsigned src_pos, unsigned n);
__isl_give isl_qpolynomial *isl_qpolynomial_project_domain_on_params(
	__isl_take isl_qpolynomial *qp);
__isl_give isl_qpolynomial *isl_qpolynomial_drop_dims(
	__isl_take isl_qpolynomial *qp,
	enum isl_dim_type type, unsigned first, unsigned n);

__isl_give isl_qpolynomial *isl_qpolynomial_substitute(
	__isl_take isl_qpolynomial *qp,
	enum isl_dim_type type, unsigned first, unsigned n,
	__isl_keep isl_qpolynomial **subs);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_qpolynomial_domain_reverse`, `isl_qpolynomial_insert_dims`, `isl_qpolynomial_add_dims`, `isl_qpolynomial_move_dims` (+3 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_qpolynomial_domain_reverse`, `isl_qpolynomial_insert_dims`, `isl_qpolynomial_add_dims`, `isl_qpolynomial_move_dims` (+3 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 98-119

````cpp
isl_stat isl_qpolynomial_as_polynomial_on_domain(__isl_keep isl_qpolynomial *qp,
	__isl_keep isl_basic_set *bset,
	isl_stat (*fn)(__isl_take isl_basic_set *bset,
		  __isl_take isl_qpolynomial *poly, void *user), void *user);

__isl_give isl_qpolynomial *isl_qpolynomial_homogenize(
	__isl_take isl_qpolynomial *poly);

__isl_give isl_qpolynomial *isl_qpolynomial_align_params(
	__isl_take isl_qpolynomial *qp, __isl_take isl_space *model);

isl_ctx *isl_term_get_ctx(__isl_keep isl_term *term);

__isl_give isl_term *isl_term_copy(__isl_keep isl_term *term);
__isl_null isl_term *isl_term_free(__isl_take isl_term *term);

isl_size isl_term_dim(__isl_keep isl_term *term, enum isl_dim_type type);
__isl_give isl_val *isl_term_get_coefficient_val(__isl_keep isl_term *term);
isl_size isl_term_get_exp(__isl_keep isl_term *term,
	enum isl_dim_type type, unsigned pos);
__isl_give isl_aff *isl_term_get_div(__isl_keep isl_term *term, unsigned pos);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_qpolynomial_as_polynomial_on_domain`, `isl_stat`, `isl_qpolynomial_homogenize`, `isl_qpolynomial_align_params` (+7 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_qpolynomial_as_polynomial_on_domain`, `isl_stat`, `isl_qpolynomial_homogenize`, `isl_qpolynomial_align_params` (+7 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 120-139

````cpp
isl_stat isl_qpolynomial_foreach_term(__isl_keep isl_qpolynomial *qp,
	isl_stat (*fn)(__isl_take isl_term *term, void *user), void *user);

__isl_give isl_val *isl_qpolynomial_eval(__isl_take isl_qpolynomial *qp,
	__isl_take isl_point *pnt);

__isl_give isl_qpolynomial *isl_qpolynomial_gist_params(
	__isl_take isl_qpolynomial *qp, __isl_take isl_set *context);
__isl_give isl_qpolynomial *isl_qpolynomial_gist(
	__isl_take isl_qpolynomial *qp, __isl_take isl_set *context);

__isl_give isl_qpolynomial *isl_qpolynomial_from_constraint(
	__isl_take isl_constraint *c, enum isl_dim_type type, unsigned pos);
__isl_give isl_qpolynomial *isl_qpolynomial_from_term(__isl_take isl_term *term);
__isl_give isl_qpolynomial *isl_qpolynomial_from_aff(__isl_take isl_aff *aff);
isl_bool isl_qpolynomial_isa_aff(__isl_keep isl_qpolynomial *qp);
__isl_give isl_aff *isl_qpolynomial_as_aff(__isl_take isl_qpolynomial *qp);
__isl_give isl_basic_map *isl_basic_map_from_qpolynomial(
	__isl_take isl_qpolynomial *qp);

````
- **EN**: This block declares or defines routines around `isl_qpolynomial_foreach_term`, `isl_stat`, `isl_qpolynomial_eval`, `isl_qpolynomial_gist_params` (+7 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_qpolynomial_foreach_term`, `isl_stat`, `isl_qpolynomial_eval`, `isl_qpolynomial_gist_params` (+7 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 140-162

````cpp
__isl_give isl_printer *isl_printer_print_qpolynomial(
	__isl_take isl_printer *p, __isl_keep isl_qpolynomial *qp);
void isl_qpolynomial_print(__isl_keep isl_qpolynomial *qp, FILE *out,
	unsigned output_format);
void isl_qpolynomial_dump(__isl_keep isl_qpolynomial *qp);

isl_ctx *isl_pw_qpolynomial_get_ctx(__isl_keep isl_pw_qpolynomial *pwqp);

isl_bool isl_pw_qpolynomial_involves_nan(__isl_keep isl_pw_qpolynomial *pwqp);
isl_bool isl_pw_qpolynomial_plain_is_equal(__isl_keep isl_pw_qpolynomial *pwqp1,
	__isl_keep isl_pw_qpolynomial *pwqp2);

__isl_give isl_pw_qpolynomial *isl_pw_qpolynomial_zero(
	__isl_take isl_space *space);
__isl_give isl_pw_qpolynomial *isl_pw_qpolynomial_alloc(__isl_take isl_set *set,
	__isl_take isl_qpolynomial *qp);
__isl_give isl_pw_qpolynomial *isl_pw_qpolynomial_from_qpolynomial(
	__isl_take isl_qpolynomial *qp);
__isl_give isl_pw_qpolynomial *isl_pw_qpolynomial_copy(
	__isl_keep isl_pw_qpolynomial *pwqp);
__isl_null isl_pw_qpolynomial *isl_pw_qpolynomial_free(
	__isl_take isl_pw_qpolynomial *pwqp);

````
- **EN**: This block declares or defines routines around `isl_printer_print_qpolynomial`, `isl_qpolynomial_print`, `isl_qpolynomial_dump`, `isl_pw_qpolynomial_get_ctx` (+7 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_printer_print_qpolynomial`, `isl_qpolynomial_print`, `isl_qpolynomial_dump`, `isl_pw_qpolynomial_get_ctx` (+7 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 163-184

````cpp
isl_bool isl_pw_qpolynomial_is_zero(__isl_keep isl_pw_qpolynomial *pwqp);

__isl_give isl_space *isl_pw_qpolynomial_get_domain_space(
	__isl_keep isl_pw_qpolynomial *pwqp);
__isl_give isl_space *isl_pw_qpolynomial_get_space(
	__isl_keep isl_pw_qpolynomial *pwqp);
__isl_give isl_pw_qpolynomial *isl_pw_qpolynomial_reset_domain_space(
	__isl_take isl_pw_qpolynomial *pwqp, __isl_take isl_space *space);
isl_size isl_pw_qpolynomial_dim(__isl_keep isl_pw_qpolynomial *pwqp,
	enum isl_dim_type type);
isl_bool isl_pw_qpolynomial_involves_param_id(
	__isl_keep isl_pw_qpolynomial *pwqp, __isl_keep isl_id *id);
isl_bool isl_pw_qpolynomial_involves_dims(__isl_keep isl_pw_qpolynomial *pwqp,
	enum isl_dim_type type, unsigned first, unsigned n);
isl_bool isl_pw_qpolynomial_has_equal_space(
	__isl_keep isl_pw_qpolynomial *pwqp1,
	__isl_keep isl_pw_qpolynomial *pwqp2);

__isl_give isl_pw_qpolynomial *isl_pw_qpolynomial_set_dim_name(
	__isl_take isl_pw_qpolynomial *pwqp,
	enum isl_dim_type type, unsigned pos, const char *s);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_pw_qpolynomial_is_zero`, `isl_pw_qpolynomial_get_domain_space`, `isl_pw_qpolynomial_get_space`, `isl_pw_qpolynomial_reset_domain_space` (+5 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_pw_qpolynomial_is_zero`, `isl_pw_qpolynomial_get_domain_space`, `isl_pw_qpolynomial_get_space`, `isl_pw_qpolynomial_reset_domain_space` (+5 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 185-205

````cpp
int isl_pw_qpolynomial_find_dim_by_name(__isl_keep isl_pw_qpolynomial *pwqp,
	enum isl_dim_type type, const char *name);

__isl_give isl_pw_qpolynomial *isl_pw_qpolynomial_reset_user(
	__isl_take isl_pw_qpolynomial *pwqp);

__isl_export
__isl_give isl_set *isl_pw_qpolynomial_domain(__isl_take isl_pw_qpolynomial *pwqp);
__isl_give isl_pw_qpolynomial *isl_pw_qpolynomial_intersect_domain(
	__isl_take isl_pw_qpolynomial *pwpq, __isl_take isl_set *set);
__isl_give isl_pw_qpolynomial *
isl_pw_qpolynomial_intersect_domain_wrapped_domain(
	__isl_take isl_pw_qpolynomial *pwpq, __isl_take isl_set *set);
__isl_give isl_pw_qpolynomial *
isl_pw_qpolynomial_intersect_domain_wrapped_range(
	__isl_take isl_pw_qpolynomial *pwpq, __isl_take isl_set *set);
__isl_give isl_pw_qpolynomial *isl_pw_qpolynomial_intersect_params(
	__isl_take isl_pw_qpolynomial *pwpq, __isl_take isl_set *set);
__isl_give isl_pw_qpolynomial *isl_pw_qpolynomial_subtract_domain(
	__isl_take isl_pw_qpolynomial *pwpq, __isl_take isl_set *set);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_pw_qpolynomial_find_dim_by_name`, `isl_pw_qpolynomial_reset_user`, `isl_pw_qpolynomial_domain`, `isl_pw_qpolynomial_intersect_domain` (+4 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_pw_qpolynomial_find_dim_by_name`, `isl_pw_qpolynomial_reset_user`, `isl_pw_qpolynomial_domain`, `isl_pw_qpolynomial_intersect_domain` (+4 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 206-229

````cpp
__isl_give isl_pw_qpolynomial *isl_pw_qpolynomial_project_domain_on_params(
	__isl_take isl_pw_qpolynomial *pwqp);
__isl_give isl_pw_qpolynomial *isl_pw_qpolynomial_from_range(
	__isl_take isl_pw_qpolynomial *pwqp);
__isl_give isl_pw_qpolynomial *isl_pw_qpolynomial_drop_dims(
	__isl_take isl_pw_qpolynomial *pwqp,
	enum isl_dim_type type, unsigned first, unsigned n);
__isl_give isl_pw_qpolynomial *isl_pw_qpolynomial_split_dims(
	__isl_take isl_pw_qpolynomial *pwqp,
	enum isl_dim_type type, unsigned first, unsigned n);

__isl_give isl_pw_qpolynomial *isl_pw_qpolynomial_drop_unused_params(
	__isl_take isl_pw_qpolynomial *pwqp);

__isl_give isl_pw_qpolynomial *isl_pw_qpolynomial_add(
	__isl_take isl_pw_qpolynomial *pwqp1,
	__isl_take isl_pw_qpolynomial *pwqp2);
__isl_give isl_pw_qpolynomial *isl_pw_qpolynomial_sub(
	__isl_take isl_pw_qpolynomial *pwqp1,
	__isl_take isl_pw_qpolynomial *pwqp2);
__isl_give isl_pw_qpolynomial *isl_pw_qpolynomial_add_disjoint(
	__isl_take isl_pw_qpolynomial *pwqp1,
	__isl_take isl_pw_qpolynomial *pwqp2);
__isl_give isl_pw_qpolynomial *isl_pw_qpolynomial_neg(
````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_pw_qpolynomial_project_domain_on_params`, `isl_pw_qpolynomial_from_range`, `isl_pw_qpolynomial_drop_dims`, `isl_pw_qpolynomial_split_dims` (+5 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_pw_qpolynomial_project_domain_on_params`, `isl_pw_qpolynomial_from_range`, `isl_pw_qpolynomial_drop_dims`, `isl_pw_qpolynomial_split_dims` (+5 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 230-253

````cpp
	__isl_take isl_pw_qpolynomial *pwqp);
__isl_give isl_pw_qpolynomial *isl_pw_qpolynomial_mul(
	__isl_take isl_pw_qpolynomial *pwqp1,
	__isl_take isl_pw_qpolynomial *pwqp2);
__isl_give isl_pw_qpolynomial *isl_pw_qpolynomial_scale_val(
	__isl_take isl_pw_qpolynomial *pwqp, __isl_take isl_val *v);
__isl_give isl_pw_qpolynomial *isl_pw_qpolynomial_scale_down_val(
	__isl_take isl_pw_qpolynomial *pwqp, __isl_take isl_val *v);
__isl_give isl_pw_qpolynomial *isl_pw_qpolynomial_pow(
	__isl_take isl_pw_qpolynomial *pwqp, unsigned exponent);

__isl_give isl_pw_qpolynomial *isl_pw_qpolynomial_domain_reverse(
	__isl_take isl_pw_qpolynomial *pwqp);
__isl_give isl_pw_qpolynomial *isl_pw_qpolynomial_insert_dims(
	__isl_take isl_pw_qpolynomial *pwqp, enum isl_dim_type type,
	unsigned first, unsigned n);
__isl_give isl_pw_qpolynomial *isl_pw_qpolynomial_add_dims(
	__isl_take isl_pw_qpolynomial *pwqp,
	enum isl_dim_type type, unsigned n);
__isl_give isl_pw_qpolynomial *isl_pw_qpolynomial_move_dims(
	__isl_take isl_pw_qpolynomial *pwqp,
	enum isl_dim_type dst_type, unsigned dst_pos,
	enum isl_dim_type src_type, unsigned src_pos, unsigned n);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_pw_qpolynomial_mul`, `isl_pw_qpolynomial_scale_val`, `isl_pw_qpolynomial_scale_down_val`, `isl_pw_qpolynomial_pow` (+4 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_pw_qpolynomial_mul`, `isl_pw_qpolynomial_scale_val`, `isl_pw_qpolynomial_scale_down_val`, `isl_pw_qpolynomial_pow` (+4 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 254-280

````cpp
__isl_give isl_pw_qpolynomial *isl_pw_qpolynomial_fix_val(
	__isl_take isl_pw_qpolynomial *pwqp,
	enum isl_dim_type type, unsigned n, __isl_take isl_val *v);

__isl_export
__isl_give isl_val *isl_pw_qpolynomial_eval(
	__isl_take isl_pw_qpolynomial *pwqp, __isl_take isl_point *pnt);

__isl_give isl_val *isl_pw_qpolynomial_max(__isl_take isl_pw_qpolynomial *pwqp);
__isl_give isl_val *isl_pw_qpolynomial_min(__isl_take isl_pw_qpolynomial *pwqp);

isl_size isl_pw_qpolynomial_n_piece(__isl_keep isl_pw_qpolynomial *pwqp);
isl_stat isl_pw_qpolynomial_foreach_piece(__isl_keep isl_pw_qpolynomial *pwqp,
	isl_stat (*fn)(__isl_take isl_set *set, __isl_take isl_qpolynomial *qp,
		    void *user), void *user);
isl_bool isl_pw_qpolynomial_every_piece(__isl_keep isl_pw_qpolynomial *pwqp,
	isl_bool (*test)(__isl_keep isl_set *set,
		__isl_keep isl_qpolynomial *qp, void *user), void *user);
isl_stat isl_pw_qpolynomial_foreach_lifted_piece(
	__isl_keep isl_pw_qpolynomial *pwqp,
	isl_stat (*fn)(__isl_take isl_set *set, __isl_take isl_qpolynomial *qp,
		    void *user), void *user);
isl_bool isl_pw_qpolynomial_isa_qpolynomial(
	__isl_keep isl_pw_qpolynomial *pwqp);
__isl_give isl_qpolynomial *isl_pw_qpolynomial_as_qpolynomial(
	__isl_take isl_pw_qpolynomial *pwqp);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_pw_qpolynomial_fix_val`, `isl_pw_qpolynomial_eval`, `isl_pw_qpolynomial_max`, `isl_pw_qpolynomial_min` (+8 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_pw_qpolynomial_fix_val`, `isl_pw_qpolynomial_eval`, `isl_pw_qpolynomial_max`, `isl_pw_qpolynomial_min` (+8 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 281-302

````cpp
__isl_give isl_pw_qpolynomial *isl_pw_qpolynomial_from_pw_aff(
	__isl_take isl_pw_aff *pwaff);

__isl_constructor
__isl_give isl_pw_qpolynomial *isl_pw_qpolynomial_read_from_str(isl_ctx *ctx,
		const char *str);
__isl_give isl_pw_qpolynomial *isl_pw_qpolynomial_read_from_file(isl_ctx *ctx,
		FILE *input);
__isl_give char *isl_pw_qpolynomial_to_str(__isl_keep isl_pw_qpolynomial *pwqp);
__isl_give isl_printer *isl_printer_print_pw_qpolynomial(
	__isl_take isl_printer *p, __isl_keep isl_pw_qpolynomial *pwqp);
void isl_pw_qpolynomial_print(__isl_keep isl_pw_qpolynomial *pwqp, FILE *out,
	unsigned output_format);
void isl_pw_qpolynomial_dump(__isl_keep isl_pw_qpolynomial *pwqp);

__isl_give isl_pw_qpolynomial *isl_pw_qpolynomial_coalesce(
	__isl_take isl_pw_qpolynomial *pwqp);
__isl_give isl_pw_qpolynomial *isl_pw_qpolynomial_gist(
	__isl_take isl_pw_qpolynomial *pwqp, __isl_take isl_set *context);
__isl_give isl_pw_qpolynomial *isl_pw_qpolynomial_gist_params(
	__isl_take isl_pw_qpolynomial *pwqp, __isl_take isl_set *context);

````
- **EN**: This block declares or defines routines around `isl_pw_qpolynomial_from_pw_aff`, `isl_pw_qpolynomial_read_from_str`, `isl_pw_qpolynomial_read_from_file`, `isl_pw_qpolynomial_to_str` (+6 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_pw_qpolynomial_from_pw_aff`, `isl_pw_qpolynomial_read_from_str`, `isl_pw_qpolynomial_read_from_file`, `isl_pw_qpolynomial_to_str` (+6 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 303-327

````cpp
__isl_give isl_pw_qpolynomial *isl_pw_qpolynomial_split_periods(
	__isl_take isl_pw_qpolynomial *pwqp, int max_periods);

__isl_give isl_pw_qpolynomial *isl_basic_set_multiplicative_call(
	__isl_take isl_basic_set *bset,
	__isl_give isl_pw_qpolynomial *(*fn)(__isl_take isl_basic_set *bset));

isl_ctx *isl_qpolynomial_fold_get_ctx(__isl_keep isl_qpolynomial_fold *fold);
enum isl_fold isl_qpolynomial_fold_get_type(__isl_keep isl_qpolynomial_fold *fold);

__isl_give isl_qpolynomial_fold *isl_qpolynomial_fold_empty(enum isl_fold type,
	__isl_take isl_space *space);
__isl_give isl_qpolynomial_fold *isl_qpolynomial_fold_alloc(
	enum isl_fold type, __isl_take isl_qpolynomial *qp);
__isl_give isl_qpolynomial_fold *isl_qpolynomial_fold_copy(
	__isl_keep isl_qpolynomial_fold *fold);
__isl_null isl_qpolynomial_fold *isl_qpolynomial_fold_free(
	__isl_take isl_qpolynomial_fold *fold);

isl_bool isl_qpolynomial_fold_is_empty(__isl_keep isl_qpolynomial_fold *fold);
isl_bool isl_qpolynomial_fold_is_nan(__isl_keep isl_qpolynomial_fold *fold);
isl_bool isl_qpolynomial_fold_plain_is_equal(
	__isl_keep isl_qpolynomial_fold *fold1,
	__isl_keep isl_qpolynomial_fold *fold2);

````
- **EN**: This block defines enum values such as `isl_fold`; declares or defines routines around `isl_pw_qpolynomial_split_periods`, `isl_basic_set_multiplicative_call`, `isl_qpolynomial_fold_get_ctx`, `isl_qpolynomial_fold_get_type` (+7 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_fold`; 声明或定义与 `isl_pw_qpolynomial_split_periods`, `isl_basic_set_multiplicative_call`, `isl_qpolynomial_fold_get_ctx`, `isl_qpolynomial_fold_get_type` (+7 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 328-351

````cpp
__isl_give isl_space *isl_qpolynomial_fold_get_domain_space(
	__isl_keep isl_qpolynomial_fold *fold);
__isl_give isl_space *isl_qpolynomial_fold_get_space(
	__isl_keep isl_qpolynomial_fold *fold);

__isl_give isl_qpolynomial_fold *isl_qpolynomial_fold_fold(
	__isl_take isl_qpolynomial_fold *fold1,
	__isl_take isl_qpolynomial_fold *fold2);

__isl_give isl_qpolynomial_fold *isl_qpolynomial_fold_scale_val(
	__isl_take isl_qpolynomial_fold *fold, __isl_take isl_val *v);
__isl_give isl_qpolynomial_fold *isl_qpolynomial_fold_scale_down_val(
	__isl_take isl_qpolynomial_fold *fold, __isl_take isl_val *v);

__isl_give isl_qpolynomial_fold *isl_qpolynomial_fold_move_dims(
	__isl_take isl_qpolynomial_fold *fold,
	enum isl_dim_type dst_type, unsigned dst_pos,
	enum isl_dim_type src_type, unsigned src_pos, unsigned n);

__isl_give isl_qpolynomial_fold *isl_qpolynomial_fold_substitute(
	__isl_take isl_qpolynomial_fold *fold,
	enum isl_dim_type type, unsigned first, unsigned n,
	__isl_keep isl_qpolynomial **subs);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_qpolynomial_fold_get_domain_space`, `isl_qpolynomial_fold_get_space`, `isl_qpolynomial_fold_fold`, `isl_qpolynomial_fold_scale_val` (+3 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_qpolynomial_fold_get_domain_space`, `isl_qpolynomial_fold_get_space`, `isl_qpolynomial_fold_fold`, `isl_qpolynomial_fold_scale_val` (+3 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 352-373

````cpp
__isl_give isl_pw_qpolynomial_fold *isl_pw_qpolynomial_fold_fix_val(
	__isl_take isl_pw_qpolynomial_fold *pwf,
	enum isl_dim_type type, unsigned n, __isl_take isl_val *v);

__isl_give isl_val *isl_qpolynomial_fold_eval(
	__isl_take isl_qpolynomial_fold *fold, __isl_take isl_point *pnt);

__isl_give isl_qpolynomial_fold *isl_qpolynomial_fold_gist_params(
	__isl_take isl_qpolynomial_fold *fold, __isl_take isl_set *context);
__isl_give isl_qpolynomial_fold *isl_qpolynomial_fold_gist(
	__isl_take isl_qpolynomial_fold *fold, __isl_take isl_set *context);

isl_stat isl_qpolynomial_fold_foreach_qpolynomial(
	__isl_keep isl_qpolynomial_fold *fold,
	isl_stat (*fn)(__isl_take isl_qpolynomial *qp, void *user), void *user);

__isl_give isl_printer *isl_printer_print_qpolynomial_fold(
	__isl_take isl_printer *p, __isl_keep isl_qpolynomial_fold *fold);
void isl_qpolynomial_fold_print(__isl_keep isl_qpolynomial_fold *fold, FILE *out,
	unsigned output_format);
void isl_qpolynomial_fold_dump(__isl_keep isl_qpolynomial_fold *fold);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_pw_qpolynomial_fold_fix_val`, `isl_qpolynomial_fold_eval`, `isl_qpolynomial_fold_gist_params`, `isl_qpolynomial_fold_gist` (+5 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_pw_qpolynomial_fold_fix_val`, `isl_qpolynomial_fold_eval`, `isl_qpolynomial_fold_gist_params`, `isl_qpolynomial_fold_gist` (+5 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 374-397

````cpp
isl_ctx *isl_pw_qpolynomial_fold_get_ctx(__isl_keep isl_pw_qpolynomial_fold *pwf);
enum isl_fold isl_pw_qpolynomial_fold_get_type(
	__isl_keep isl_pw_qpolynomial_fold *pwf);

isl_bool isl_pw_qpolynomial_fold_involves_nan(
	__isl_keep isl_pw_qpolynomial_fold *pwf);
isl_bool isl_pw_qpolynomial_fold_plain_is_equal(
	__isl_keep isl_pw_qpolynomial_fold *pwf1,
	__isl_keep isl_pw_qpolynomial_fold *pwf2);

__isl_give isl_pw_qpolynomial_fold *isl_pw_qpolynomial_fold_from_pw_qpolynomial(
	enum isl_fold type, __isl_take isl_pw_qpolynomial *pwqp);

__isl_give isl_pw_qpolynomial_fold *isl_pw_qpolynomial_fold_alloc(
	enum isl_fold type,
	__isl_take isl_set *set, __isl_take isl_qpolynomial_fold *fold);
__isl_give isl_pw_qpolynomial_fold *
isl_pw_qpolynomial_fold_from_qpolynomial_fold(
	__isl_take isl_qpolynomial_fold *fold);
__isl_give isl_pw_qpolynomial_fold *isl_pw_qpolynomial_fold_copy(
	__isl_keep isl_pw_qpolynomial_fold *pwf);
__isl_null isl_pw_qpolynomial_fold *isl_pw_qpolynomial_fold_free(
	__isl_take isl_pw_qpolynomial_fold *pwf);

````
- **EN**: This block defines enum values such as `isl_fold`; declares or defines routines around `isl_pw_qpolynomial_fold_get_ctx`, `isl_pw_qpolynomial_fold_get_type`, `isl_pw_qpolynomial_fold_involves_nan`, `isl_pw_qpolynomial_fold_plain_is_equal` (+5 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_fold`; 声明或定义与 `isl_pw_qpolynomial_fold_get_ctx`, `isl_pw_qpolynomial_fold_get_type`, `isl_pw_qpolynomial_fold_involves_nan`, `isl_pw_qpolynomial_fold_plain_is_equal` (+5 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 398-419

````cpp
isl_bool isl_pw_qpolynomial_fold_is_zero(
	__isl_keep isl_pw_qpolynomial_fold *pwf);

__isl_give isl_space *isl_pw_qpolynomial_fold_get_domain_space(
	__isl_keep isl_pw_qpolynomial_fold *pwf);
__isl_give isl_space *isl_pw_qpolynomial_fold_get_space(
	__isl_keep isl_pw_qpolynomial_fold *pwf);
__isl_give isl_pw_qpolynomial_fold *isl_pw_qpolynomial_fold_reset_space(
	__isl_take isl_pw_qpolynomial_fold *pwf, __isl_take isl_space *space);
isl_size isl_pw_qpolynomial_fold_dim(__isl_keep isl_pw_qpolynomial_fold *pwf,
	enum isl_dim_type type);
isl_bool isl_pw_qpolynomial_fold_involves_param_id(
	__isl_keep isl_pw_qpolynomial_fold *pwf, __isl_keep isl_id *id);
isl_bool isl_pw_qpolynomial_fold_has_equal_space(
	__isl_keep isl_pw_qpolynomial_fold *pwf1,
	__isl_keep isl_pw_qpolynomial_fold *pwf2);

size_t isl_pw_qpolynomial_fold_size(__isl_keep isl_pw_qpolynomial_fold *pwf);

__isl_give isl_pw_qpolynomial_fold *isl_pw_qpolynomial_fold_zero(
	__isl_take isl_space *space, enum isl_fold type);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_pw_qpolynomial_fold_is_zero`, `isl_pw_qpolynomial_fold_get_domain_space`, `isl_pw_qpolynomial_fold_get_space`, `isl_pw_qpolynomial_fold_reset_space` (+5 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_pw_qpolynomial_fold_is_zero`, `isl_pw_qpolynomial_fold_get_domain_space`, `isl_pw_qpolynomial_fold_get_space`, `isl_pw_qpolynomial_fold_reset_space` (+5 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 420-445

````cpp
__isl_give isl_pw_qpolynomial_fold *isl_pw_qpolynomial_fold_set_dim_name(
	__isl_take isl_pw_qpolynomial_fold *pwf,
	enum isl_dim_type type, unsigned pos, const char *s);

int isl_pw_qpolynomial_fold_find_dim_by_name(
	__isl_keep isl_pw_qpolynomial_fold *pwf,
	enum isl_dim_type type, const char *name);

__isl_give isl_pw_qpolynomial_fold *isl_pw_qpolynomial_fold_reset_user(
	__isl_take isl_pw_qpolynomial_fold *pwf);

__isl_give isl_set *isl_pw_qpolynomial_fold_domain(
	__isl_take isl_pw_qpolynomial_fold *pwf);
__isl_give isl_pw_qpolynomial_fold *isl_pw_qpolynomial_fold_intersect_domain(
	__isl_take isl_pw_qpolynomial_fold *pwf, __isl_take isl_set *set);
__isl_give isl_pw_qpolynomial_fold *
isl_pw_qpolynomial_fold_intersect_domain_wrapped_domain(
	__isl_take isl_pw_qpolynomial_fold *pwf, __isl_take isl_set *set);
__isl_give isl_pw_qpolynomial_fold *
isl_pw_qpolynomial_fold_intersect_domain_wrapped_range(
	__isl_take isl_pw_qpolynomial_fold *pwf, __isl_take isl_set *set);
__isl_give isl_pw_qpolynomial_fold *isl_pw_qpolynomial_fold_intersect_params(
	__isl_take isl_pw_qpolynomial_fold *pwf, __isl_take isl_set *set);
__isl_give isl_pw_qpolynomial_fold *isl_pw_qpolynomial_fold_subtract_domain(
	__isl_take isl_pw_qpolynomial_fold *pwf, __isl_take isl_set *set);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_pw_qpolynomial_fold_set_dim_name`, `isl_pw_qpolynomial_fold_find_dim_by_name`, `isl_pw_qpolynomial_fold_reset_user`, `isl_pw_qpolynomial_fold_domain` (+5 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_pw_qpolynomial_fold_set_dim_name`, `isl_pw_qpolynomial_fold_find_dim_by_name`, `isl_pw_qpolynomial_fold_reset_user`, `isl_pw_qpolynomial_fold_domain` (+5 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 446-471

````cpp
__isl_give isl_pw_qpolynomial_fold *isl_pw_qpolynomial_fold_add(
	__isl_take isl_pw_qpolynomial_fold *pwf1,
	__isl_take isl_pw_qpolynomial_fold *pwf2);
__isl_give isl_pw_qpolynomial_fold *isl_pw_qpolynomial_fold_fold(
	__isl_take isl_pw_qpolynomial_fold *pwf1,
	__isl_take isl_pw_qpolynomial_fold *pwf2);
__isl_give isl_pw_qpolynomial_fold *isl_pw_qpolynomial_fold_add_disjoint(
	__isl_take isl_pw_qpolynomial_fold *pwf1,
	__isl_take isl_pw_qpolynomial_fold *pwf2);
__isl_give isl_pw_qpolynomial_fold *isl_pw_qpolynomial_fold_scale_val(
	__isl_take isl_pw_qpolynomial_fold *pwf, __isl_take isl_val *v);
__isl_give isl_pw_qpolynomial_fold *isl_pw_qpolynomial_fold_scale_down_val(
	__isl_take isl_pw_qpolynomial_fold *pwf, __isl_take isl_val *v);

__isl_give isl_pw_qpolynomial_fold *isl_pw_qpolynomial_fold_project_domain_on_params(
	__isl_take isl_pw_qpolynomial_fold *pwf);
__isl_give isl_pw_qpolynomial_fold *isl_pw_qpolynomial_fold_from_range(
	__isl_take isl_pw_qpolynomial_fold *pwf);
__isl_give isl_pw_qpolynomial_fold *isl_pw_qpolynomial_fold_drop_dims(
	__isl_take isl_pw_qpolynomial_fold *pwf,
	enum isl_dim_type type, unsigned first, unsigned n);
__isl_give isl_pw_qpolynomial_fold *isl_pw_qpolynomial_fold_move_dims(
	__isl_take isl_pw_qpolynomial_fold *pwf,
	enum isl_dim_type dst_type, unsigned dst_pos,
	enum isl_dim_type src_type, unsigned src_pos, unsigned n);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_pw_qpolynomial_fold_add`, `isl_pw_qpolynomial_fold_fold`, `isl_pw_qpolynomial_fold_add_disjoint`, `isl_pw_qpolynomial_fold_scale_val` (+5 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_pw_qpolynomial_fold_add`, `isl_pw_qpolynomial_fold_fold`, `isl_pw_qpolynomial_fold_add_disjoint`, `isl_pw_qpolynomial_fold_scale_val` (+5 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 472-496

````cpp
__isl_give isl_pw_qpolynomial_fold *isl_pw_qpolynomial_fold_drop_unused_params(
	__isl_take isl_pw_qpolynomial_fold *pwf);

__isl_give isl_val *isl_pw_qpolynomial_fold_eval(
	__isl_take isl_pw_qpolynomial_fold *pwf, __isl_take isl_point *pnt);

isl_size isl_pw_qpolynomial_fold_n_piece(
	__isl_keep isl_pw_qpolynomial_fold *pwf);
isl_stat isl_pw_qpolynomial_fold_foreach_piece(
	__isl_keep isl_pw_qpolynomial_fold *pwf,
	isl_stat (*fn)(__isl_take isl_set *set,
		__isl_take isl_qpolynomial_fold *fold, void *user), void *user);
isl_bool isl_pw_qpolynomial_fold_every_piece(
	__isl_keep isl_pw_qpolynomial_fold *pwf,
	isl_bool (*test)(__isl_keep isl_set *set,
		__isl_keep isl_qpolynomial_fold *fold, void *user), void *user);
isl_stat isl_pw_qpolynomial_fold_foreach_lifted_piece(
	__isl_keep isl_pw_qpolynomial_fold *pwf,
	isl_stat (*fn)(__isl_take isl_set *set,
		__isl_take isl_qpolynomial_fold *fold, void *user), void *user);
isl_bool isl_pw_qpolynomial_fold_isa_qpolynomial_fold(
	__isl_keep isl_pw_qpolynomial_fold *pwf);
__isl_give isl_qpolynomial_fold *isl_pw_qpolynomial_fold_as_qpolynomial_fold(
	__isl_take isl_pw_qpolynomial_fold *pwf);

````
- **EN**: This block declares or defines routines around `isl_pw_qpolynomial_fold_drop_unused_params`, `isl_pw_qpolynomial_fold_eval`, `isl_pw_qpolynomial_fold_n_piece`, `isl_pw_qpolynomial_fold_foreach_piece` (+6 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_pw_qpolynomial_fold_drop_unused_params`, `isl_pw_qpolynomial_fold_eval`, `isl_pw_qpolynomial_fold_n_piece`, `isl_pw_qpolynomial_fold_foreach_piece` (+6 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 497-516

````cpp
__isl_give isl_pw_qpolynomial_fold *isl_pw_qpolynomial_fold_read_from_str(
	isl_ctx *ctx, const char *str);
__isl_give isl_printer *isl_printer_print_pw_qpolynomial_fold(
	__isl_take isl_printer *p, __isl_keep isl_pw_qpolynomial_fold *pwf);
void isl_pw_qpolynomial_fold_print(__isl_keep isl_pw_qpolynomial_fold *pwf,
	FILE *out, unsigned output_format);
void isl_pw_qpolynomial_fold_dump(__isl_keep isl_pw_qpolynomial_fold *pwf);

__isl_give isl_pw_qpolynomial_fold *isl_pw_qpolynomial_fold_coalesce(
	__isl_take isl_pw_qpolynomial_fold *pwf);
__isl_give isl_pw_qpolynomial_fold *isl_pw_qpolynomial_fold_gist(
	__isl_take isl_pw_qpolynomial_fold *pwf, __isl_take isl_set *context);
__isl_give isl_pw_qpolynomial_fold *isl_pw_qpolynomial_fold_gist_params(
	__isl_take isl_pw_qpolynomial_fold *pwf, __isl_take isl_set *context);

__isl_give isl_val *isl_pw_qpolynomial_fold_max(
	__isl_take isl_pw_qpolynomial_fold *pwf);
__isl_give isl_val *isl_pw_qpolynomial_fold_min(
	__isl_take isl_pw_qpolynomial_fold *pwf);

````
- **EN**: This block declares or defines routines around `isl_pw_qpolynomial_fold_read_from_str`, `isl_printer_print_pw_qpolynomial_fold`, `isl_pw_qpolynomial_fold_print`, `isl_pw_qpolynomial_fold_dump` (+5 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_pw_qpolynomial_fold_read_from_str`, `isl_printer_print_pw_qpolynomial_fold`, `isl_pw_qpolynomial_fold_print`, `isl_pw_qpolynomial_fold_dump` (+5 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 517-537

````cpp
__isl_give isl_pw_qpolynomial_fold *isl_pw_qpolynomial_bound(
	__isl_take isl_pw_qpolynomial *pwqp, enum isl_fold type,
	isl_bool *tight);
__isl_give isl_pw_qpolynomial_fold *isl_pw_qpolynomial_fold_bound(
	__isl_take isl_pw_qpolynomial_fold *pwf, isl_bool *tight);
__isl_give isl_pw_qpolynomial_fold *isl_set_apply_pw_qpolynomial_fold(
	__isl_take isl_set *set, __isl_take isl_pw_qpolynomial_fold *pwf,
	isl_bool *tight);
__isl_give isl_pw_qpolynomial_fold *isl_map_apply_pw_qpolynomial_fold(
	__isl_take isl_map *map, __isl_take isl_pw_qpolynomial_fold *pwf,
	isl_bool *tight);

__isl_give isl_pw_qpolynomial *isl_pw_qpolynomial_to_polynomial(
	__isl_take isl_pw_qpolynomial *pwqp, int sign);

isl_ctx *isl_union_pw_qpolynomial_get_ctx(
	__isl_keep isl_union_pw_qpolynomial *upwqp);

isl_size isl_union_pw_qpolynomial_dim(
	__isl_keep isl_union_pw_qpolynomial *upwqp, enum isl_dim_type type);

````
- **EN**: This block declares or defines routines around `isl_pw_qpolynomial_bound`, `isl_pw_qpolynomial_fold_bound`, `isl_set_apply_pw_qpolynomial_fold`, `isl_map_apply_pw_qpolynomial_fold` (+3 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_pw_qpolynomial_bound`, `isl_pw_qpolynomial_fold_bound`, `isl_set_apply_pw_qpolynomial_fold`, `isl_map_apply_pw_qpolynomial_fold` (+3 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 538-560

````cpp
isl_bool isl_union_pw_qpolynomial_involves_nan(
	__isl_keep isl_union_pw_qpolynomial *upwqp);
isl_bool isl_union_pw_qpolynomial_plain_is_equal(
	__isl_keep isl_union_pw_qpolynomial *upwqp1,
	__isl_keep isl_union_pw_qpolynomial *upwqp2);

__isl_give isl_union_pw_qpolynomial *isl_pw_qpolynomial_to_union_pw_qpolynomial(
	__isl_take isl_pw_qpolynomial *pwqp);
__isl_give isl_union_pw_qpolynomial *isl_union_pw_qpolynomial_from_pw_qpolynomial(__isl_take isl_pw_qpolynomial *pwqp);
__isl_give isl_union_pw_qpolynomial *isl_union_pw_qpolynomial_zero_ctx(
	isl_ctx *ctx);
__isl_give isl_union_pw_qpolynomial *isl_union_pw_qpolynomial_zero_space(
	__isl_take isl_space *space);
__isl_give isl_union_pw_qpolynomial *isl_union_pw_qpolynomial_zero(
	__isl_take isl_space *space);
__isl_give isl_union_pw_qpolynomial *isl_union_pw_qpolynomial_add_pw_qpolynomial(
	__isl_take isl_union_pw_qpolynomial *upwqp,
	__isl_take isl_pw_qpolynomial *pwqp);
__isl_give isl_union_pw_qpolynomial *isl_union_pw_qpolynomial_copy(
	__isl_keep isl_union_pw_qpolynomial *upwqp);
__isl_null isl_union_pw_qpolynomial *isl_union_pw_qpolynomial_free(
	__isl_take isl_union_pw_qpolynomial *upwqp);

````
- **EN**: This block declares or defines routines around `isl_union_pw_qpolynomial_involves_nan`, `isl_union_pw_qpolynomial_plain_is_equal`, `isl_pw_qpolynomial_to_union_pw_qpolynomial`, `isl_union_pw_qpolynomial_from_pw_qpolynomial` (+6 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_union_pw_qpolynomial_involves_nan`, `isl_union_pw_qpolynomial_plain_is_equal`, `isl_pw_qpolynomial_to_union_pw_qpolynomial`, `isl_union_pw_qpolynomial_from_pw_qpolynomial` (+6 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 561-583

````cpp
__isl_constructor
__isl_give isl_union_pw_qpolynomial *isl_union_pw_qpolynomial_read_from_str(
	isl_ctx *ctx, const char *str);
__isl_give char *isl_union_pw_qpolynomial_to_str(
	__isl_keep isl_union_pw_qpolynomial *upwqp);

__isl_give isl_union_pw_qpolynomial *isl_union_pw_qpolynomial_neg(
	__isl_take isl_union_pw_qpolynomial *upwqp);

__isl_give isl_union_pw_qpolynomial *isl_union_pw_qpolynomial_add(
	__isl_take isl_union_pw_qpolynomial *upwqp1,
	__isl_take isl_union_pw_qpolynomial *upwqp2);
__isl_give isl_union_pw_qpolynomial *isl_union_pw_qpolynomial_sub(
	__isl_take isl_union_pw_qpolynomial *upwqp1,
	__isl_take isl_union_pw_qpolynomial *upwqp2);
__isl_give isl_union_pw_qpolynomial *isl_union_pw_qpolynomial_mul(
	__isl_take isl_union_pw_qpolynomial *upwqp1,
	__isl_take isl_union_pw_qpolynomial *upwqp2);
__isl_give isl_union_pw_qpolynomial *isl_union_pw_qpolynomial_scale_val(
	__isl_take isl_union_pw_qpolynomial *upwqp, __isl_take isl_val *v);
__isl_give isl_union_pw_qpolynomial *isl_union_pw_qpolynomial_scale_down_val(
	__isl_take isl_union_pw_qpolynomial *upwqp, __isl_take isl_val *v);

````
- **EN**: This block declares or defines routines around `isl_union_pw_qpolynomial_read_from_str`, `isl_union_pw_qpolynomial_to_str`, `isl_union_pw_qpolynomial_neg`, `isl_union_pw_qpolynomial_add` (+4 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_union_pw_qpolynomial_read_from_str`, `isl_union_pw_qpolynomial_to_str`, `isl_union_pw_qpolynomial_neg`, `isl_union_pw_qpolynomial_add` (+4 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 584-607

````cpp
__isl_export
__isl_give isl_union_set *isl_union_pw_qpolynomial_domain(
	__isl_take isl_union_pw_qpolynomial *upwqp);
__isl_give isl_union_pw_qpolynomial *
isl_union_pw_qpolynomial_intersect_domain_space(
	__isl_take isl_union_pw_qpolynomial *upwpq,
	__isl_take isl_space *space);
__isl_give isl_union_pw_qpolynomial *
isl_union_pw_qpolynomial_intersect_domain_union_set(
	__isl_take isl_union_pw_qpolynomial *upwpq,
	__isl_take isl_union_set *uset);
__isl_give isl_union_pw_qpolynomial *isl_union_pw_qpolynomial_intersect_domain(
	__isl_take isl_union_pw_qpolynomial *upwpq,
	__isl_take isl_union_set *uset);
__isl_give isl_union_pw_qpolynomial *
isl_union_pw_qpolynomial_intersect_domain_wrapped_domain(
	__isl_take isl_union_pw_qpolynomial *upwpq,
	__isl_take isl_union_set *uset);
__isl_give isl_union_pw_qpolynomial *
isl_union_pw_qpolynomial_intersect_domain_wrapped_range(
	__isl_take isl_union_pw_qpolynomial *upwpq,
	__isl_take isl_union_set *uset);
__isl_give isl_union_pw_qpolynomial *isl_union_pw_qpolynomial_intersect_params(
	__isl_take isl_union_pw_qpolynomial *upwpq,
````
- **EN**: This block declares or defines routines around `isl_union_pw_qpolynomial_domain`, `isl_union_pw_qpolynomial_intersect_domain_space`, `isl_union_pw_qpolynomial_intersect_domain_union_set`, `isl_union_pw_qpolynomial_intersect_domain` (+3 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_union_pw_qpolynomial_domain`, `isl_union_pw_qpolynomial_intersect_domain_space`, `isl_union_pw_qpolynomial_intersect_domain_union_set`, `isl_union_pw_qpolynomial_intersect_domain` (+3 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 608-630

````cpp
	__isl_take isl_set *set);
__isl_give isl_union_pw_qpolynomial *
isl_union_pw_qpolynomial_subtract_domain_union_set(
	__isl_take isl_union_pw_qpolynomial *upwpq,
	__isl_take isl_union_set *uset);
__isl_give isl_union_pw_qpolynomial *
isl_union_pw_qpolynomial_subtract_domain_space(
	__isl_take isl_union_pw_qpolynomial *upwpq,
	__isl_take isl_space *space);
__isl_give isl_union_pw_qpolynomial *isl_union_pw_qpolynomial_subtract_domain(
	__isl_take isl_union_pw_qpolynomial *upwpq,
	__isl_take isl_union_set *uset);

__isl_give isl_space *isl_union_pw_qpolynomial_get_space(
	__isl_keep isl_union_pw_qpolynomial *upwqp);
__isl_give isl_pw_qpolynomial_list *
isl_union_pw_qpolynomial_get_pw_qpolynomial_list(
	__isl_keep isl_union_pw_qpolynomial *upwqp);

__isl_give isl_union_pw_qpolynomial *isl_union_pw_qpolynomial_set_dim_name(
	__isl_take isl_union_pw_qpolynomial *upwqp,
	enum isl_dim_type type, unsigned pos, const char *s);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_union_pw_qpolynomial_subtract_domain_union_set`, `isl_union_pw_qpolynomial_subtract_domain_space`, `isl_union_pw_qpolynomial_subtract_domain`, `isl_union_pw_qpolynomial_get_space` (+2 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_union_pw_qpolynomial_subtract_domain_union_set`, `isl_union_pw_qpolynomial_subtract_domain_space`, `isl_union_pw_qpolynomial_subtract_domain`, `isl_union_pw_qpolynomial_get_space` (+2 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 631-655

````cpp
int isl_union_pw_qpolynomial_find_dim_by_name(
	__isl_keep isl_union_pw_qpolynomial *upwqp,
	enum isl_dim_type type, const char *name);

__isl_give isl_union_pw_qpolynomial *isl_union_pw_qpolynomial_domain_reverse(
	__isl_take isl_union_pw_qpolynomial *upwqp);
__isl_give isl_union_pw_qpolynomial *isl_union_pw_qpolynomial_drop_dims(
	__isl_take isl_union_pw_qpolynomial *upwqp,
	enum isl_dim_type type, unsigned first, unsigned n);
__isl_give isl_union_pw_qpolynomial *isl_union_pw_qpolynomial_reset_user(
	__isl_take isl_union_pw_qpolynomial *upwqp);

__isl_export
__isl_give isl_val *isl_union_pw_qpolynomial_eval(
	__isl_take isl_union_pw_qpolynomial *upwqp, __isl_take isl_point *pnt);

__isl_give isl_union_pw_qpolynomial *isl_union_pw_qpolynomial_coalesce(
	__isl_take isl_union_pw_qpolynomial *upwqp);
__isl_give isl_union_pw_qpolynomial *isl_union_pw_qpolynomial_gist(
	__isl_take isl_union_pw_qpolynomial *upwqp,
	__isl_take isl_union_set *context);
__isl_give isl_union_pw_qpolynomial *isl_union_pw_qpolynomial_gist_params(
	__isl_take isl_union_pw_qpolynomial *upwqp,
	__isl_take isl_set *context);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_union_pw_qpolynomial_find_dim_by_name`, `isl_union_pw_qpolynomial_domain_reverse`, `isl_union_pw_qpolynomial_drop_dims`, `isl_union_pw_qpolynomial_reset_user` (+4 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_union_pw_qpolynomial_find_dim_by_name`, `isl_union_pw_qpolynomial_domain_reverse`, `isl_union_pw_qpolynomial_drop_dims`, `isl_union_pw_qpolynomial_reset_user` (+4 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 656-676

````cpp
__isl_give isl_union_pw_qpolynomial *isl_union_pw_qpolynomial_align_params(
	__isl_take isl_union_pw_qpolynomial *upwqp,
	__isl_take isl_space *model);
__isl_give isl_union_pw_qpolynomial *
isl_union_pw_qpolynomial_drop_unused_params(
	__isl_take isl_union_pw_qpolynomial *upwqp);

isl_size isl_union_pw_qpolynomial_n_pw_qpolynomial(
	__isl_keep isl_union_pw_qpolynomial *upwqp);
isl_stat isl_union_pw_qpolynomial_foreach_pw_qpolynomial(
	__isl_keep isl_union_pw_qpolynomial *upwqp,
	isl_stat (*fn)(__isl_take isl_pw_qpolynomial *pwqp, void *user),
	void *user);
isl_bool isl_union_pw_qpolynomial_every_pw_qpolynomial(
	__isl_keep isl_union_pw_qpolynomial *upwqp,
	isl_bool (*test)(__isl_keep isl_pw_qpolynomial *pwqp, void *user),
	void *user);
__isl_give isl_pw_qpolynomial *isl_union_pw_qpolynomial_extract_pw_qpolynomial(
	__isl_keep isl_union_pw_qpolynomial *upwqp,
	__isl_take isl_space *space);

````
- **EN**: This block declares or defines routines around `isl_union_pw_qpolynomial_align_params`, `isl_union_pw_qpolynomial_drop_unused_params`, `isl_union_pw_qpolynomial_n_pw_qpolynomial`, `isl_union_pw_qpolynomial_foreach_pw_qpolynomial` (+4 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_union_pw_qpolynomial_align_params`, `isl_union_pw_qpolynomial_drop_unused_params`, `isl_union_pw_qpolynomial_n_pw_qpolynomial`, `isl_union_pw_qpolynomial_foreach_pw_qpolynomial` (+4 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 677-700

````cpp
__isl_give isl_printer *isl_printer_print_union_pw_qpolynomial(
	__isl_take isl_printer *p, __isl_keep isl_union_pw_qpolynomial *upwqp);

isl_ctx *isl_union_pw_qpolynomial_fold_get_ctx(
	__isl_keep isl_union_pw_qpolynomial_fold *upwf);

isl_size isl_union_pw_qpolynomial_fold_dim(
	__isl_keep isl_union_pw_qpolynomial_fold *upwf, enum isl_dim_type type);

isl_bool isl_union_pw_qpolynomial_fold_involves_nan(
	__isl_keep isl_union_pw_qpolynomial_fold *upwf);
isl_bool isl_union_pw_qpolynomial_fold_plain_is_equal(
	__isl_keep isl_union_pw_qpolynomial_fold *upwf1,
	__isl_keep isl_union_pw_qpolynomial_fold *upwf2);

__isl_give isl_union_pw_qpolynomial_fold *
isl_pw_qpolynomial_fold_to_union_pw_qpolynomial_fold(
	__isl_take isl_pw_qpolynomial_fold *pwf);
__isl_give isl_union_pw_qpolynomial_fold *isl_union_pw_qpolynomial_fold_from_pw_qpolynomial_fold(__isl_take isl_pw_qpolynomial_fold *pwf);
__isl_give isl_union_pw_qpolynomial_fold *
isl_union_pw_qpolynomial_fold_zero_ctx(isl_ctx *ctx, enum isl_fold type);
__isl_give isl_union_pw_qpolynomial_fold *
isl_union_pw_qpolynomial_fold_zero_space(__isl_take isl_space *space,
	enum isl_fold type);
````
- **EN**: This block defines enum values such as `isl_fold`; declares or defines routines around `isl_printer_print_union_pw_qpolynomial`, `isl_union_pw_qpolynomial_fold_get_ctx`, `isl_union_pw_qpolynomial_fold_dim`, `isl_union_pw_qpolynomial_fold_involves_nan` (+5 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_fold`; 声明或定义与 `isl_printer_print_union_pw_qpolynomial`, `isl_union_pw_qpolynomial_fold_get_ctx`, `isl_union_pw_qpolynomial_fold_dim`, `isl_union_pw_qpolynomial_fold_involves_nan` (+5 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 701-723

````cpp
__isl_give isl_union_pw_qpolynomial_fold *isl_union_pw_qpolynomial_fold_zero(
	__isl_take isl_space *space, enum isl_fold type);
__isl_give isl_union_pw_qpolynomial_fold *isl_union_pw_qpolynomial_fold_fold_pw_qpolynomial_fold(
	__isl_take isl_union_pw_qpolynomial_fold *upwqp,
	__isl_take isl_pw_qpolynomial_fold *pwqp);
__isl_null isl_union_pw_qpolynomial_fold *isl_union_pw_qpolynomial_fold_free(
	__isl_take isl_union_pw_qpolynomial_fold *upwf);
__isl_give isl_union_pw_qpolynomial_fold *isl_union_pw_qpolynomial_fold_copy(
	__isl_keep isl_union_pw_qpolynomial_fold *upwf);

__isl_give isl_union_pw_qpolynomial_fold *isl_union_pw_qpolynomial_fold_fold(
	__isl_take isl_union_pw_qpolynomial_fold *upwf1,
	__isl_take isl_union_pw_qpolynomial_fold *upwf2);
__isl_give isl_union_pw_qpolynomial_fold *isl_union_pw_qpolynomial_fold_add_union_pw_qpolynomial(
	__isl_take isl_union_pw_qpolynomial_fold *upwf,
	__isl_take isl_union_pw_qpolynomial *upwqp);
__isl_give isl_union_pw_qpolynomial_fold *
isl_union_pw_qpolynomial_fold_scale_val(
	__isl_take isl_union_pw_qpolynomial_fold *upwf, __isl_take isl_val *v);
__isl_give isl_union_pw_qpolynomial_fold *
isl_union_pw_qpolynomial_fold_scale_down_val(
	__isl_take isl_union_pw_qpolynomial_fold *upwf, __isl_take isl_val *v);

````
- **EN**: This block declares or defines routines around `isl_union_pw_qpolynomial_fold_zero`, `isl_union_pw_qpolynomial_fold_fold_pw_qpolynomial_fold`, `isl_union_pw_qpolynomial_fold_free`, `isl_union_pw_qpolynomial_fold_copy` (+4 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_union_pw_qpolynomial_fold_zero`, `isl_union_pw_qpolynomial_fold_fold_pw_qpolynomial_fold`, `isl_union_pw_qpolynomial_fold_free`, `isl_union_pw_qpolynomial_fold_copy` (+4 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 724-747

````cpp
__isl_give isl_union_set *isl_union_pw_qpolynomial_fold_domain(
	__isl_take isl_union_pw_qpolynomial_fold *upwf);
__isl_give isl_union_pw_qpolynomial_fold *
isl_union_pw_qpolynomial_fold_intersect_domain_space(
	__isl_take isl_union_pw_qpolynomial_fold *upwf,
	__isl_take isl_space *space);
__isl_give isl_union_pw_qpolynomial_fold *
isl_union_pw_qpolynomial_fold_intersect_domain_union_set(
	__isl_take isl_union_pw_qpolynomial_fold *upwf,
	__isl_take isl_union_set *uset);
__isl_give isl_union_pw_qpolynomial_fold *isl_union_pw_qpolynomial_fold_intersect_domain(
	__isl_take isl_union_pw_qpolynomial_fold *upwf,
	__isl_take isl_union_set *uset);
__isl_give isl_union_pw_qpolynomial_fold *
isl_union_pw_qpolynomial_fold_intersect_domain_wrapped_domain(
	__isl_take isl_union_pw_qpolynomial_fold *upwf,
	__isl_take isl_union_set *uset);
__isl_give isl_union_pw_qpolynomial_fold *
isl_union_pw_qpolynomial_fold_intersect_domain_wrapped_range(
	__isl_take isl_union_pw_qpolynomial_fold *upwf,
	__isl_take isl_union_set *uset);
__isl_give isl_union_pw_qpolynomial_fold *
isl_union_pw_qpolynomial_fold_intersect_params(
	__isl_take isl_union_pw_qpolynomial_fold *upwf,
````
- **EN**: This block declares or defines routines around `isl_union_pw_qpolynomial_fold_domain`, `isl_union_pw_qpolynomial_fold_intersect_domain_space`, `isl_union_pw_qpolynomial_fold_intersect_domain_union_set`, `isl_union_pw_qpolynomial_fold_intersect_domain` (+3 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_union_pw_qpolynomial_fold_domain`, `isl_union_pw_qpolynomial_fold_intersect_domain_space`, `isl_union_pw_qpolynomial_fold_intersect_domain_union_set`, `isl_union_pw_qpolynomial_fold_intersect_domain` (+3 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 748-769

````cpp
	__isl_take isl_set *set);
__isl_give isl_union_pw_qpolynomial_fold *
isl_union_pw_qpolynomial_fold_subtract_domain_union_set(
	__isl_take isl_union_pw_qpolynomial_fold *upwf,
	__isl_take isl_union_set *uset);
__isl_give isl_union_pw_qpolynomial_fold *
isl_union_pw_qpolynomial_fold_subtract_domain_space(
	__isl_take isl_union_pw_qpolynomial_fold *upwf,
	__isl_take isl_space *space);
__isl_give isl_union_pw_qpolynomial_fold *
isl_union_pw_qpolynomial_fold_subtract_domain(
	__isl_take isl_union_pw_qpolynomial_fold *upwf,
	__isl_take isl_union_set *uset);

enum isl_fold isl_union_pw_qpolynomial_fold_get_type(
	__isl_keep isl_union_pw_qpolynomial_fold *upwf);
__isl_give isl_space *isl_union_pw_qpolynomial_fold_get_space(
	__isl_keep isl_union_pw_qpolynomial_fold *upwf);
__isl_give isl_pw_qpolynomial_fold_list *
isl_union_pw_qpolynomial_fold_get_pw_qpolynomial_fold_list(
	__isl_keep isl_union_pw_qpolynomial_fold *upwf);

````
- **EN**: This block defines enum values such as `isl_fold`; declares or defines routines around `isl_union_pw_qpolynomial_fold_subtract_domain_union_set`, `isl_union_pw_qpolynomial_fold_subtract_domain_space`, `isl_union_pw_qpolynomial_fold_subtract_domain`, `isl_union_pw_qpolynomial_fold_get_type` (+2 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_fold`; 声明或定义与 `isl_union_pw_qpolynomial_fold_subtract_domain_union_set`, `isl_union_pw_qpolynomial_fold_subtract_domain_space`, `isl_union_pw_qpolynomial_fold_subtract_domain`, `isl_union_pw_qpolynomial_fold_get_type` (+2 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 770-790

````cpp
__isl_give isl_union_pw_qpolynomial_fold *
isl_union_pw_qpolynomial_fold_set_dim_name(
	__isl_take isl_union_pw_qpolynomial_fold *upwf,
	enum isl_dim_type type, unsigned pos, const char *s);

int isl_union_pw_qpolynomial_fold_find_dim_by_name(
	__isl_keep isl_union_pw_qpolynomial_fold *upwf,
	enum isl_dim_type type, const char *name);

__isl_give isl_union_pw_qpolynomial_fold *
	isl_union_pw_qpolynomial_fold_drop_dims(
	__isl_take isl_union_pw_qpolynomial_fold *upwf,
	enum isl_dim_type type, unsigned first, unsigned n);
__isl_give isl_union_pw_qpolynomial_fold *
isl_union_pw_qpolynomial_fold_reset_user(
	__isl_take isl_union_pw_qpolynomial_fold *upwf);

__isl_give isl_val *isl_union_pw_qpolynomial_fold_eval(
	__isl_take isl_union_pw_qpolynomial_fold *upwf,
	__isl_take isl_point *pnt);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_union_pw_qpolynomial_fold_set_dim_name`, `isl_union_pw_qpolynomial_fold_find_dim_by_name`, `isl_union_pw_qpolynomial_fold_drop_dims`, `isl_union_pw_qpolynomial_fold_reset_user` (+1 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_union_pw_qpolynomial_fold_set_dim_name`, `isl_union_pw_qpolynomial_fold_find_dim_by_name`, `isl_union_pw_qpolynomial_fold_drop_dims`, `isl_union_pw_qpolynomial_fold_reset_user` (+1 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 791-814

````cpp
__isl_give isl_union_pw_qpolynomial_fold *isl_union_pw_qpolynomial_fold_coalesce(
	__isl_take isl_union_pw_qpolynomial_fold *upwf);
__isl_give isl_union_pw_qpolynomial_fold *isl_union_pw_qpolynomial_fold_gist(
	__isl_take isl_union_pw_qpolynomial_fold *upwf,
	__isl_take isl_union_set *context);
__isl_give isl_union_pw_qpolynomial_fold *
isl_union_pw_qpolynomial_fold_gist_params(
	__isl_take isl_union_pw_qpolynomial_fold *upwf,
	__isl_take isl_set *context);

__isl_give isl_union_pw_qpolynomial_fold *isl_union_pw_qpolynomial_fold_align_params(
	__isl_take isl_union_pw_qpolynomial_fold *upwf,
	__isl_take isl_space *model);
__isl_give isl_union_pw_qpolynomial_fold *
isl_union_pw_qpolynomial_fold_drop_unused_params(
	__isl_take isl_union_pw_qpolynomial_fold *upwf);

isl_size isl_union_pw_qpolynomial_fold_n_pw_qpolynomial_fold(
	__isl_keep isl_union_pw_qpolynomial_fold *upwf);
isl_stat isl_union_pw_qpolynomial_fold_foreach_pw_qpolynomial_fold(
	__isl_keep isl_union_pw_qpolynomial_fold *upwf,
	isl_stat (*fn)(__isl_take isl_pw_qpolynomial_fold *pwf,
		    void *user), void *user);
isl_bool isl_union_pw_qpolynomial_fold_every_pw_qpolynomial_fold(
````
- **EN**: This block declares or defines routines around `isl_union_pw_qpolynomial_fold_coalesce`, `isl_union_pw_qpolynomial_fold_gist`, `isl_union_pw_qpolynomial_fold_gist_params`, `isl_union_pw_qpolynomial_fold_align_params` (+5 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_union_pw_qpolynomial_fold_coalesce`, `isl_union_pw_qpolynomial_fold_gist`, `isl_union_pw_qpolynomial_fold_gist_params`, `isl_union_pw_qpolynomial_fold_align_params` (+5 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 815-835

````cpp
	__isl_keep isl_union_pw_qpolynomial_fold *upwf,
	isl_bool (*test)(__isl_keep isl_pw_qpolynomial_fold *pwf,
		void *user), void *user);
__isl_give isl_pw_qpolynomial_fold *isl_union_pw_qpolynomial_fold_extract_pw_qpolynomial_fold(
	__isl_keep isl_union_pw_qpolynomial_fold *upwf,
	__isl_take isl_space *space);

__isl_give isl_printer *isl_printer_print_union_pw_qpolynomial_fold(
	__isl_take isl_printer *p,
	__isl_keep isl_union_pw_qpolynomial_fold *upwf);

__isl_give isl_union_pw_qpolynomial_fold *isl_union_pw_qpolynomial_bound(
	__isl_take isl_union_pw_qpolynomial *upwqp,
	enum isl_fold type, isl_bool *tight);
__isl_give isl_union_pw_qpolynomial_fold *isl_union_set_apply_union_pw_qpolynomial_fold(
	__isl_take isl_union_set *uset,
	__isl_take isl_union_pw_qpolynomial_fold *upwf, isl_bool *tight);
__isl_give isl_union_pw_qpolynomial_fold *isl_union_map_apply_union_pw_qpolynomial_fold(
	__isl_take isl_union_map *umap,
	__isl_take isl_union_pw_qpolynomial_fold *upwf, isl_bool *tight);

````
- **EN**: This block defines enum values such as `isl_fold`; declares or defines routines around `isl_bool`, `isl_union_pw_qpolynomial_fold_extract_pw_qpolynomial_fold`, `isl_printer_print_union_pw_qpolynomial_fold`, `isl_union_pw_qpolynomial_bound` (+2 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_fold`; 声明或定义与 `isl_bool`, `isl_union_pw_qpolynomial_fold_extract_pw_qpolynomial_fold`, `isl_printer_print_union_pw_qpolynomial_fold`, `isl_union_pw_qpolynomial_bound` (+2 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 836-847

````cpp
__isl_give isl_union_pw_qpolynomial *isl_union_pw_qpolynomial_to_polynomial(
	__isl_take isl_union_pw_qpolynomial *upwqp, int sign);

ISL_DECLARE_LIST_FN(qpolynomial)
ISL_DECLARE_LIST_FN(pw_qpolynomial)
ISL_DECLARE_LIST_FN(pw_qpolynomial_fold)

#if defined(__cplusplus)
}
#endif

#endif
````
- **EN**: This block uses preprocessor directives to control compilation, macros, or include guards; declares or defines routines around `isl_union_pw_qpolynomial_to_polynomial`, `ISL_DECLARE_LIST_FN`; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 使用预处理指令控制编译、宏或头文件保护; 声明或定义与 `isl_union_pw_qpolynomial_to_polynomial`, `ISL_DECLARE_LIST_FN` 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

## Key Concepts / 关键概念

- **ISL relations and sets**
  - **CN**: ISL 关系与集合
- **Public interface declarations**
  - **CN**: 公共接口声明

## Dependencies / 依赖关系

- **ISL headers**: `isl/ctx.h`, `isl/constraint.h`, `isl/space_type.h`, `isl/set_type.h`, `isl/point.h`, `isl/printer.h`, `isl/union_set_type.h`, `isl/aff_type.h` (+2 more) — ISL relation/set primitives that support polyhedral analysis and transformations.
  **ISL headers（CN）**：`isl/ctx.h`, `isl/constraint.h`, `isl/space_type.h`, `isl/set_type.h`, `isl/point.h`, `isl/printer.h`, `isl/union_set_type.h`, `isl/aff_type.h` (+2 more) —— 支撑多面体分析与变换的 ISL 关系/集合原语。
