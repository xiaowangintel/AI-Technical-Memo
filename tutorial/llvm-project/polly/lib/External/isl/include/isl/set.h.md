# set.h — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `polly/lib/External/isl/include/isl/set.h` | `polly/lib/External/isl/include/isl/set.h` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | Declares upstream ISL interfaces and C++ wrappers that Polly depends on for polyhedral operations. | 声明 Polly 依赖的上游 ISL 接口与 C++ 包装层，用于多面体计算。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````cpp
/*
 * Copyright 2008-2009 Katholieke Universiteit Leuven
 *
 * Use of this software is governed by the MIT license
 *
 * Written by Sven Verdoolaege, K.U.Leuven, Departement
 * Computerwetenschappen, Celestijnenlaan 200A, B-3001 Leuven, Belgium
 */

#ifndef ISL_SET_H
#define ISL_SET_H

#include <isl/id_type.h>
#include <isl/map_type.h>
#include <isl/aff_type.h>
#include <isl/list.h>
#include <isl/mat.h>
#include <isl/point.h>
#include <isl/local_space.h>
#include <isl/val_type.h>
#include <isl/stdint.h>
#include <isl/stride_info.h>
#include <isl/fixed_box.h>

````
- **EN**: This block imports ISL headers needed by the surrounding code; uses preprocessor directives to control compilation, macros, or include guards; defines macros like `ISL_SET_H`; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 引入周边逻辑所需的 ISL 头文件; 使用预处理指令控制编译、宏或头文件保护; 定义宏，例如 `ISL_SET_H`; 保留用于解释意图、用法或算法背景的注释.

### Lines 25-48

````cpp
#if defined(__cplusplus)
extern "C" {
#endif

isl_size isl_basic_set_n_dim(__isl_keep isl_basic_set *bset);
isl_size isl_basic_set_n_param(__isl_keep isl_basic_set *bset);
isl_size isl_basic_set_total_dim(__isl_keep const isl_basic_set *bset);
isl_size isl_basic_set_dim(__isl_keep isl_basic_set *bset,
				enum isl_dim_type type);

isl_size isl_set_n_dim(__isl_keep isl_set *set);
isl_size isl_set_n_param(__isl_keep isl_set *set);
__isl_export
isl_size isl_set_tuple_dim(__isl_keep isl_set *set);
isl_size isl_set_dim(__isl_keep isl_set *set, enum isl_dim_type type);

isl_ctx *isl_basic_set_get_ctx(__isl_keep isl_basic_set *bset);
isl_ctx *isl_set_get_ctx(__isl_keep isl_set *set);
__isl_give isl_space *isl_basic_set_get_space(__isl_keep isl_basic_set *bset);
__isl_export
__isl_give isl_space *isl_set_get_space(__isl_keep isl_set *set);
__isl_give isl_set *isl_set_reset_space(__isl_take isl_set *set,
	__isl_take isl_space *space);

````
- **EN**: This block uses preprocessor directives to control compilation, macros, or include guards; defines enum values such as `isl_dim_type`; declares or defines routines around `isl_basic_set_n_dim`, `isl_basic_set_n_param`, `isl_basic_set_total_dim`, `isl_basic_set_dim` (+9 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 使用预处理指令控制编译、宏或头文件保护; 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_basic_set_n_dim`, `isl_basic_set_n_param`, `isl_basic_set_total_dim`, `isl_basic_set_dim` (+9 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 49-73

````cpp
__isl_give isl_aff *isl_basic_set_get_div(__isl_keep isl_basic_set *bset,
	int pos);

__isl_give isl_local_space *isl_basic_set_get_local_space(
	__isl_keep isl_basic_set *bset);

const char *isl_basic_set_get_tuple_name(__isl_keep isl_basic_set *bset);
isl_bool isl_set_has_tuple_name(__isl_keep isl_set *set);
const char *isl_set_get_tuple_name(__isl_keep isl_set *set);
__isl_give isl_basic_set *isl_basic_set_set_tuple_name(
	__isl_take isl_basic_set *set, const char *s);
__isl_give isl_set *isl_set_set_tuple_name(__isl_take isl_set *set,
	const char *s);
const char *isl_basic_set_get_dim_name(__isl_keep isl_basic_set *bset,
	enum isl_dim_type type, unsigned pos);
__isl_give isl_basic_set *isl_basic_set_set_dim_name(
	__isl_take isl_basic_set *bset,
	enum isl_dim_type type, unsigned pos, const char *s);
isl_bool isl_set_has_dim_name(__isl_keep isl_set *set,
	enum isl_dim_type type, unsigned pos);
const char *isl_set_get_dim_name(__isl_keep isl_set *set,
	enum isl_dim_type type, unsigned pos);
__isl_give isl_set *isl_set_set_dim_name(__isl_take isl_set *set,
	enum isl_dim_type type, unsigned pos, const char *s);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_basic_set_get_div`, `isl_basic_set_get_local_space`, `isl_basic_set_get_tuple_name`, `isl_set_has_tuple_name` (+8 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_basic_set_get_div`, `isl_basic_set_get_local_space`, `isl_basic_set_get_tuple_name`, `isl_set_has_tuple_name` (+8 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 74-95

````cpp
__isl_give isl_id *isl_basic_set_get_dim_id(__isl_keep isl_basic_set *bset,
	enum isl_dim_type type, unsigned pos);
__isl_give isl_basic_set *isl_basic_set_set_tuple_id(
	__isl_take isl_basic_set *bset, __isl_take isl_id *id);
__isl_give isl_set *isl_set_set_dim_id(__isl_take isl_set *set,
	enum isl_dim_type type, unsigned pos, __isl_take isl_id *id);
isl_bool isl_set_has_dim_id(__isl_keep isl_set *set,
	enum isl_dim_type type, unsigned pos);
__isl_give isl_id *isl_set_get_dim_id(__isl_keep isl_set *set,
	enum isl_dim_type type, unsigned pos);
__isl_give isl_set *isl_set_set_tuple_id(__isl_take isl_set *set,
	__isl_take isl_id *id);
__isl_give isl_set *isl_set_reset_tuple_id(__isl_take isl_set *set);
isl_bool isl_set_has_tuple_id(__isl_keep isl_set *set);
__isl_give isl_id *isl_set_get_tuple_id(__isl_keep isl_set *set);
__isl_give isl_set *isl_set_reset_user(__isl_take isl_set *set);

int isl_set_find_dim_by_id(__isl_keep isl_set *set, enum isl_dim_type type,
	__isl_keep isl_id *id);
int isl_set_find_dim_by_name(__isl_keep isl_set *set, enum isl_dim_type type,
	const char *name);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_basic_set_get_dim_id`, `isl_basic_set_set_tuple_id`, `isl_set_set_dim_id`, `isl_set_has_dim_id` (+8 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_basic_set_get_dim_id`, `isl_basic_set_set_tuple_id`, `isl_set_set_dim_id`, `isl_set_has_dim_id` (+8 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 96-119

````cpp
int isl_basic_set_is_rational(__isl_keep isl_basic_set *bset);

__isl_null isl_basic_set *isl_basic_set_free(__isl_take isl_basic_set *bset);
__isl_give isl_basic_set *isl_basic_set_copy(__isl_keep isl_basic_set *bset);
__isl_give isl_basic_set *isl_basic_set_empty(__isl_take isl_space *space);
__isl_give isl_basic_set *isl_basic_set_universe(__isl_take isl_space *space);
__isl_give isl_basic_set *isl_basic_set_nat_universe(
	__isl_take isl_space *space);
__isl_give isl_basic_set *isl_basic_set_positive_orthant(
	__isl_take isl_space *space);
void isl_basic_set_print_internal(__isl_keep isl_basic_set *bset,
				FILE *out, int indent);
__isl_export
__isl_give isl_basic_set *isl_basic_set_intersect(
		__isl_take isl_basic_set *bset1,
		__isl_take isl_basic_set *bset2);
__isl_export
__isl_give isl_basic_set *isl_basic_set_intersect_params(
	__isl_take isl_basic_set *bset1, __isl_take isl_basic_set *bset2);
__isl_export
__isl_give isl_basic_set *isl_basic_set_apply(
		__isl_take isl_basic_set *bset,
		__isl_take isl_basic_map *bmap);
__isl_give isl_basic_set *isl_basic_set_preimage_multi_aff(
````
- **EN**: This block declares or defines routines around `isl_basic_set_is_rational`, `isl_basic_set_free`, `isl_basic_set_copy`, `isl_basic_set_empty` (+8 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_basic_set_is_rational`, `isl_basic_set_free`, `isl_basic_set_copy`, `isl_basic_set_empty` (+8 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 120-139

````cpp
	__isl_take isl_basic_set *bset, __isl_take isl_multi_aff *ma);
__isl_export
__isl_give isl_basic_set *isl_basic_set_affine_hull(
		__isl_take isl_basic_set *bset);
__isl_give isl_basic_set *isl_basic_set_remove_dims(
	__isl_take isl_basic_set *bset,
	enum isl_dim_type type, unsigned first, unsigned n);
__isl_export
__isl_give isl_basic_set *isl_basic_set_sample(__isl_take isl_basic_set *bset);
__isl_export
__isl_give isl_basic_set *isl_basic_set_detect_equalities(
						__isl_take isl_basic_set *bset);
__isl_give isl_basic_set *isl_basic_set_remove_redundancies(
	__isl_take isl_basic_set *bset);
__isl_give isl_set *isl_set_remove_redundancies(__isl_take isl_set *set);
__isl_give isl_basic_set *isl_basic_set_list_intersect(
	__isl_take struct isl_basic_set_list *list);

__isl_give isl_set *isl_set_list_union(__isl_take isl_set_list *list);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_basic_set_affine_hull`, `isl_basic_set_remove_dims`, `isl_basic_set_sample`, `isl_basic_set_detect_equalities` (+4 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_basic_set_affine_hull`, `isl_basic_set_remove_dims`, `isl_basic_set_sample`, `isl_basic_set_detect_equalities` (+4 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 140-163

````cpp
__isl_give isl_basic_set *isl_basic_set_read_from_file(isl_ctx *ctx,
	FILE *input);
__isl_constructor
__isl_give isl_basic_set *isl_basic_set_read_from_str(isl_ctx *ctx,
	const char *str);
__isl_give isl_set *isl_set_read_from_file(isl_ctx *ctx, FILE *input);
__isl_constructor
__isl_give isl_set *isl_set_read_from_str(isl_ctx *ctx, const char *str);
void isl_basic_set_dump(__isl_keep isl_basic_set *bset);
void isl_set_dump(__isl_keep isl_set *set);
__isl_give isl_printer *isl_printer_print_basic_set(
	__isl_take isl_printer *printer, __isl_keep isl_basic_set *bset);
__isl_give isl_printer *isl_printer_print_set(__isl_take isl_printer *printer,
	__isl_keep isl_set *map);
__isl_give isl_basic_set *isl_basic_set_fix_si(__isl_take isl_basic_set *bset,
		enum isl_dim_type type, unsigned pos, int value);
__isl_give isl_basic_set *isl_basic_set_fix_val(__isl_take isl_basic_set *bset,
	enum isl_dim_type type, unsigned pos, __isl_take isl_val *v);
__isl_give isl_set *isl_set_fix_si(__isl_take isl_set *set,
		enum isl_dim_type type, unsigned pos, int value);
__isl_give isl_set *isl_set_lower_bound_si(__isl_take isl_set *set,
		enum isl_dim_type type, unsigned pos, int value);
__isl_give isl_basic_set *isl_basic_set_lower_bound_val(
	__isl_take isl_basic_set *bset, enum isl_dim_type type, unsigned pos,
````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_basic_set_read_from_file`, `isl_basic_set_read_from_str`, `isl_set_read_from_file`, `isl_set_read_from_str` (+9 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_basic_set_read_from_file`, `isl_basic_set_read_from_str`, `isl_set_read_from_file`, `isl_set_read_from_str` (+9 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 164-186

````cpp
	__isl_take isl_val *value);
__isl_give isl_set *isl_set_lower_bound_val(__isl_take isl_set *set,
	enum isl_dim_type type, unsigned pos, __isl_take isl_val *value);
__isl_give isl_set *isl_set_upper_bound_si(__isl_take isl_set *set,
	enum isl_dim_type type, unsigned pos, int value);
__isl_give isl_basic_set *isl_basic_set_upper_bound_val(
	__isl_take isl_basic_set *bset, enum isl_dim_type type, unsigned pos,
	__isl_take isl_val *value);
__isl_give isl_set *isl_set_upper_bound_val(__isl_take isl_set *set,
	enum isl_dim_type type, unsigned pos, __isl_take isl_val *value);
__isl_overload
__isl_give isl_set *isl_set_lower_bound_multi_val(__isl_take isl_set *set,
	__isl_take isl_multi_val *lower);
__isl_overload
__isl_give isl_set *isl_set_upper_bound_multi_val(__isl_take isl_set *set,
	__isl_take isl_multi_val *upper);
__isl_overload
__isl_give isl_set *isl_set_lower_bound_multi_pw_aff(__isl_take isl_set *set,
	__isl_take isl_multi_pw_aff *lower);
__isl_overload
__isl_give isl_set *isl_set_upper_bound_multi_pw_aff(__isl_take isl_set *set,
	__isl_take isl_multi_pw_aff *upper);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_set_lower_bound_val`, `isl_set_upper_bound_si`, `isl_basic_set_upper_bound_val`, `isl_set_upper_bound_val` (+4 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_set_lower_bound_val`, `isl_set_upper_bound_si`, `isl_basic_set_upper_bound_val`, `isl_set_upper_bound_val` (+4 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 187-210

````cpp
__isl_give isl_set *isl_set_equate(__isl_take isl_set *set,
	enum isl_dim_type type1, int pos1, enum isl_dim_type type2, int pos2);

__isl_export
isl_bool isl_basic_set_is_equal(__isl_keep isl_basic_set *bset1,
	__isl_keep isl_basic_set *bset2);
isl_bool isl_basic_set_is_disjoint(__isl_keep isl_basic_set *bset1,
	__isl_keep isl_basic_set *bset2);

__isl_give isl_set *isl_basic_set_partial_lexmin(
		__isl_take isl_basic_set *bset, __isl_take isl_basic_set *dom,
		__isl_give isl_set **empty);
__isl_give isl_set *isl_basic_set_partial_lexmax(
		__isl_take isl_basic_set *bset, __isl_take isl_basic_set *dom,
		__isl_give isl_set **empty);
__isl_give isl_set *isl_set_partial_lexmin(
		__isl_take isl_set *set, __isl_take isl_set *dom,
		__isl_give isl_set **empty);
__isl_give isl_set *isl_set_partial_lexmax(
		__isl_take isl_set *set, __isl_take isl_set *dom,
		__isl_give isl_set **empty);
__isl_export
__isl_give isl_set *isl_basic_set_lexmin(__isl_take isl_basic_set *bset);
__isl_export
````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_set_equate`, `isl_basic_set_is_equal`, `isl_basic_set_is_disjoint`, `isl_basic_set_partial_lexmin` (+4 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_set_equate`, `isl_basic_set_is_equal`, `isl_basic_set_is_disjoint`, `isl_basic_set_partial_lexmin` (+4 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 211-232

````cpp
__isl_give isl_set *isl_basic_set_lexmax(__isl_take isl_basic_set *bset);
__isl_export
__isl_give isl_set *isl_set_lexmin(__isl_take isl_set *set);
__isl_export
__isl_give isl_set *isl_set_lexmax(__isl_take isl_set *set);
__isl_give isl_pw_multi_aff *isl_basic_set_partial_lexmin_pw_multi_aff(
	__isl_take isl_basic_set *bset, __isl_take isl_basic_set *dom,
	__isl_give isl_set **empty);
__isl_give isl_pw_multi_aff *isl_basic_set_partial_lexmax_pw_multi_aff(
	__isl_take isl_basic_set *bset, __isl_take isl_basic_set *dom,
	__isl_give isl_set **empty);
__isl_export
__isl_give isl_pw_multi_aff *isl_set_lexmin_pw_multi_aff(
	__isl_take isl_set *set);
__isl_export
__isl_give isl_pw_multi_aff *isl_set_lexmax_pw_multi_aff(
	__isl_take isl_set *set);
__isl_export
__isl_give isl_multi_pw_aff *isl_set_min_multi_pw_aff(__isl_take isl_set *set);
__isl_export
__isl_give isl_multi_pw_aff *isl_set_max_multi_pw_aff(__isl_take isl_set *set);

````
- **EN**: This block declares or defines routines around `isl_basic_set_lexmax`, `isl_set_lexmin`, `isl_set_lexmax`, `isl_basic_set_partial_lexmin_pw_multi_aff` (+5 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_basic_set_lexmax`, `isl_set_lexmin`, `isl_set_lexmax`, `isl_basic_set_partial_lexmin_pw_multi_aff` (+5 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 233-260

````cpp
__isl_export
__isl_give isl_set *isl_basic_set_union(
		__isl_take isl_basic_set *bset1,
		__isl_take isl_basic_set *bset2);

int isl_basic_set_compare_at(__isl_keep isl_basic_set *bset1,
	__isl_keep isl_basic_set *bset2, int pos);
int isl_set_follows_at(__isl_keep isl_set *set1,
	__isl_keep isl_set *set2, int pos);

__isl_export
__isl_give isl_basic_set *isl_basic_set_params(__isl_take isl_basic_set *bset);
__isl_give isl_basic_set *isl_basic_set_from_params(
	__isl_take isl_basic_set *bset);
__isl_export
__isl_give isl_set *isl_set_params(__isl_take isl_set *set);
__isl_give isl_set *isl_set_from_params(__isl_take isl_set *set);

__isl_export
__isl_give isl_set *isl_set_bind(__isl_take isl_set *set,
	__isl_take isl_multi_id *tuple);
__isl_export
__isl_give isl_set *isl_set_unbind_params(__isl_take isl_set *set,
	__isl_take isl_multi_id *tuple);
__isl_export
__isl_give isl_map *isl_set_unbind_params_insert_domain(
	__isl_take isl_set *set, __isl_take isl_multi_id *domain);

````
- **EN**: This block declares or defines routines around `isl_basic_set_union`, `isl_basic_set_compare_at`, `isl_set_follows_at`, `isl_basic_set_params` (+6 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_basic_set_union`, `isl_basic_set_compare_at`, `isl_set_follows_at`, `isl_basic_set_params` (+6 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 261-284

````cpp
isl_stat isl_basic_set_dims_get_sign(__isl_keep isl_basic_set *bset,
	enum isl_dim_type type, unsigned pos, unsigned n, int *signs);

isl_bool isl_basic_set_plain_is_universe(__isl_keep isl_basic_set *bset);
isl_bool isl_basic_set_is_universe(__isl_keep isl_basic_set *bset);
isl_bool isl_basic_set_plain_is_empty(__isl_keep isl_basic_set *bset);
__isl_export
isl_bool isl_basic_set_is_empty(__isl_keep isl_basic_set *bset);
isl_bool isl_basic_set_is_bounded(__isl_keep isl_basic_set *bset);
__isl_export
isl_bool isl_basic_set_is_subset(__isl_keep isl_basic_set *bset1,
	__isl_keep isl_basic_set *bset2);
isl_bool isl_basic_set_plain_is_equal(__isl_keep isl_basic_set *bset1,
	__isl_keep isl_basic_set *bset2);

__isl_export
__isl_give isl_set *isl_set_empty(__isl_take isl_space *space);
__isl_export
__isl_give isl_set *isl_set_universe(__isl_take isl_space *space);
__isl_export
__isl_give isl_set *isl_space_universe_set(__isl_take isl_space *space);
__isl_give isl_set *isl_set_nat_universe(__isl_take isl_space *space);
__isl_give isl_set *isl_set_copy(__isl_keep isl_set *set);
__isl_null isl_set *isl_set_free(__isl_take isl_set *set);
````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_basic_set_dims_get_sign`, `isl_basic_set_plain_is_universe`, `isl_basic_set_is_universe`, `isl_basic_set_plain_is_empty` (+10 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_basic_set_dims_get_sign`, `isl_basic_set_plain_is_universe`, `isl_basic_set_is_universe`, `isl_basic_set_plain_is_empty` (+10 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 285-311

````cpp
__isl_export
__isl_give isl_set *isl_basic_set_to_set(__isl_take isl_basic_set *bset);
__isl_constructor
__isl_give isl_set *isl_set_from_basic_set(__isl_take isl_basic_set *bset);
__isl_export
__isl_give isl_basic_set *isl_set_sample(__isl_take isl_set *set);
__isl_export
__isl_give isl_point *isl_basic_set_sample_point(__isl_take isl_basic_set *bset);
__isl_export
__isl_give isl_point *isl_set_sample_point(__isl_take isl_set *set);
__isl_export
__isl_give isl_set *isl_set_detect_equalities(__isl_take isl_set *set);
__isl_export
__isl_give isl_basic_set *isl_set_affine_hull(__isl_take isl_set *set);
__isl_give isl_basic_set *isl_set_convex_hull(__isl_take isl_set *set);
__isl_export
__isl_give isl_basic_set *isl_set_polyhedral_hull(__isl_take isl_set *set);
__isl_give isl_basic_set *isl_set_simple_hull(__isl_take isl_set *set);
__isl_export
__isl_give isl_basic_set *isl_set_unshifted_simple_hull(
	__isl_take isl_set *set);
__isl_give isl_basic_set *isl_set_plain_unshifted_simple_hull(
	__isl_take isl_set *set);
__isl_give isl_basic_set *isl_set_unshifted_simple_hull_from_set_list(
	__isl_take isl_set *set, __isl_take isl_set_list *list);
__isl_give isl_basic_set *isl_set_bounded_simple_hull(__isl_take isl_set *set);

````
- **EN**: This block declares or defines routines around `isl_basic_set_to_set`, `isl_set_from_basic_set`, `isl_set_sample`, `isl_basic_set_sample_point` (+10 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_basic_set_to_set`, `isl_set_from_basic_set`, `isl_set_sample`, `isl_basic_set_sample_point` (+10 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 312-335

````cpp
__isl_export
__isl_give isl_set *isl_set_wrapped_reverse(__isl_take isl_set *set);
__isl_give isl_set *isl_set_union_disjoint(
	__isl_take isl_set *set1, __isl_take isl_set *set2);
__isl_export
__isl_give isl_set *isl_set_union(
		__isl_take isl_set *set1,
		__isl_take isl_set *set2);
__isl_export
__isl_give isl_set *isl_set_product(__isl_take isl_set *set1,
	__isl_take isl_set *set2);
__isl_give isl_basic_set *isl_basic_set_flat_product(
	__isl_take isl_basic_set *bset1, __isl_take isl_basic_set *bset2);
__isl_give isl_set *isl_set_flat_product(__isl_take isl_set *set1,
	__isl_take isl_set *set2);
__isl_export
__isl_give isl_set *isl_set_intersect(
		__isl_take isl_set *set1,
		__isl_take isl_set *set2);
__isl_export
__isl_give isl_set *isl_set_intersect_params(__isl_take isl_set *set,
		__isl_take isl_set *params);
__isl_give isl_set *isl_set_intersect_factor_domain(__isl_take isl_set *set,
	__isl_take isl_set *domain);
````
- **EN**: This block declares or defines routines around `isl_set_wrapped_reverse`, `isl_set_union_disjoint`, `isl_set_union`, `isl_set_product` (+5 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_set_wrapped_reverse`, `isl_set_union_disjoint`, `isl_set_union`, `isl_set_product` (+5 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 336-359

````cpp
__isl_give isl_set *isl_set_intersect_factor_range(__isl_take isl_set *set,
	__isl_take isl_set *range);
__isl_export
__isl_give isl_set *isl_set_subtract(
		__isl_take isl_set *set1,
		__isl_take isl_set *set2);
__isl_export
__isl_give isl_set *isl_set_complement(__isl_take isl_set *set);
__isl_export
__isl_give isl_set *isl_set_apply(
		__isl_take isl_set *set,
		__isl_take isl_map *map);
__isl_overload
__isl_give isl_set *isl_set_preimage_multi_aff(__isl_take isl_set *set,
	__isl_take isl_multi_aff *ma);
__isl_overload
__isl_give isl_set *isl_set_preimage_pw_multi_aff(__isl_take isl_set *set,
	__isl_take isl_pw_multi_aff *pma);
__isl_overload
__isl_give isl_set *isl_set_preimage_multi_pw_aff(__isl_take isl_set *set,
	__isl_take isl_multi_pw_aff *mpa);
__isl_give isl_set *isl_set_fix_val(__isl_take isl_set *set,
	enum isl_dim_type type, unsigned pos, __isl_take isl_val *v);
__isl_give isl_set *isl_set_fix_dim_si(__isl_take isl_set *set,
````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_set_intersect_factor_range`, `isl_set_subtract`, `isl_set_complement`, `isl_set_apply` (+5 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_set_intersect_factor_range`, `isl_set_subtract`, `isl_set_complement`, `isl_set_apply` (+5 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 360-383

````cpp
		unsigned dim, int value);
__isl_give isl_basic_set *isl_basic_set_insert_dims(
	__isl_take isl_basic_set *bset,
	enum isl_dim_type type, unsigned pos, unsigned n);
__isl_give isl_set *isl_set_insert_dims(__isl_take isl_set *set,
		enum isl_dim_type type, unsigned pos, unsigned n);
__isl_give isl_basic_set *isl_basic_set_add_dims(__isl_take isl_basic_set *bset,
		enum isl_dim_type type, unsigned n);
__isl_give isl_set *isl_set_add_dims(__isl_take isl_set *set,
		enum isl_dim_type type, unsigned n);
__isl_give isl_basic_set *isl_basic_set_move_dims(__isl_take isl_basic_set *bset,
	enum isl_dim_type dst_type, unsigned dst_pos,
	enum isl_dim_type src_type, unsigned src_pos, unsigned n);
__isl_give isl_set *isl_set_move_dims(__isl_take isl_set *set,
	enum isl_dim_type dst_type, unsigned dst_pos,
	enum isl_dim_type src_type, unsigned src_pos, unsigned n);
__isl_give isl_basic_set *isl_basic_set_project_out(
		__isl_take isl_basic_set *bset,
		enum isl_dim_type type, unsigned first, unsigned n);
__isl_overload
__isl_give isl_set *isl_set_project_out_param_id(__isl_take isl_set *set,
	__isl_take isl_id *id);
__isl_overload
__isl_give isl_set *isl_set_project_out_param_id_list(__isl_take isl_set *set,
````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_basic_set_insert_dims`, `isl_set_insert_dims`, `isl_basic_set_add_dims`, `isl_set_add_dims` (+5 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_basic_set_insert_dims`, `isl_set_insert_dims`, `isl_basic_set_add_dims`, `isl_set_add_dims` (+5 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 384-407

````cpp
	__isl_take isl_id_list *list);
__isl_give isl_set *isl_set_project_out(__isl_take isl_set *set,
		enum isl_dim_type type, unsigned first, unsigned n);
__isl_export
__isl_give isl_set *isl_set_project_out_all_params(__isl_take isl_set *set);
__isl_give isl_map *isl_set_project_onto_map(__isl_take isl_set *set,
	enum isl_dim_type type, unsigned first, unsigned n);
__isl_give isl_basic_set *isl_basic_set_remove_divs(
	__isl_take isl_basic_set *bset);
__isl_give isl_basic_set *isl_basic_set_eliminate(
	__isl_take isl_basic_set *bset,
	enum isl_dim_type type, unsigned first, unsigned n);
__isl_give isl_set *isl_set_eliminate(__isl_take isl_set *set,
	enum isl_dim_type type, unsigned first, unsigned n);
__isl_give isl_set *isl_set_eliminate_dims(__isl_take isl_set *set,
		unsigned first, unsigned n);
__isl_give isl_set *isl_set_remove_dims(__isl_take isl_set *bset,
	enum isl_dim_type type, unsigned first, unsigned n);
__isl_give isl_basic_set *isl_basic_set_remove_divs_involving_dims(
	__isl_take isl_basic_set *bset,
	enum isl_dim_type type, unsigned first, unsigned n);
__isl_give isl_set *isl_set_remove_divs_involving_dims(__isl_take isl_set *set,
	enum isl_dim_type type, unsigned first, unsigned n);
__isl_give isl_basic_set *isl_basic_set_remove_unknown_divs(
````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_set_project_out`, `isl_set_project_out_all_params`, `isl_set_project_onto_map`, `isl_basic_set_remove_divs` (+7 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_set_project_out`, `isl_set_project_out_all_params`, `isl_set_project_onto_map`, `isl_basic_set_remove_divs` (+7 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 408-429

````cpp
	__isl_take isl_basic_set *bset);
__isl_give isl_set *isl_set_remove_unknown_divs(__isl_take isl_set *set);
__isl_give isl_set *isl_set_remove_divs(__isl_take isl_set *set);
__isl_give isl_set *isl_set_split_dims(__isl_take isl_set *set,
	enum isl_dim_type type, unsigned first, unsigned n);

__isl_give isl_basic_set *isl_basic_set_drop_constraints_involving_dims(
	__isl_take isl_basic_set *bset,
	enum isl_dim_type type, unsigned first, unsigned n);
__isl_give isl_basic_set *isl_basic_set_drop_constraints_not_involving_dims(
	__isl_take isl_basic_set *bset,
	enum isl_dim_type type, unsigned first, unsigned n);
__isl_give isl_set *isl_set_drop_constraints_involving_dims(
	__isl_take isl_set *set,
	enum isl_dim_type type, unsigned first, unsigned n);
__isl_give isl_set *isl_set_drop_constraints_not_involving_dims(
	__isl_take isl_set *set,
	enum isl_dim_type type, unsigned first, unsigned n);

__isl_export
isl_bool isl_set_involves_locals(__isl_keep isl_set *set);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_set_remove_unknown_divs`, `isl_set_remove_divs`, `isl_set_split_dims`, `isl_basic_set_drop_constraints_involving_dims` (+4 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_set_remove_unknown_divs`, `isl_set_remove_divs`, `isl_set_split_dims`, `isl_basic_set_drop_constraints_involving_dims` (+4 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 430-457

````cpp
isl_bool isl_basic_set_involves_dims(__isl_keep isl_basic_set *bset,
	enum isl_dim_type type, unsigned first, unsigned n);
isl_bool isl_set_involves_dims(__isl_keep isl_set *set,
	enum isl_dim_type type, unsigned first, unsigned n);

void isl_set_print_internal(__isl_keep isl_set *set, FILE *out, int indent);
isl_bool isl_set_plain_is_empty(__isl_keep isl_set *set);
isl_bool isl_set_plain_is_universe(__isl_keep isl_set *set);
isl_bool isl_set_is_params(__isl_keep isl_set *set);
__isl_export
isl_bool isl_set_is_empty(__isl_keep isl_set *set);
isl_bool isl_set_is_bounded(__isl_keep isl_set *set);
__isl_export
isl_bool isl_set_is_subset(__isl_keep isl_set *set1, __isl_keep isl_set *set2);
__isl_export
isl_bool isl_set_is_strict_subset(__isl_keep isl_set *set1,
	__isl_keep isl_set *set2);
__isl_export
isl_bool isl_set_is_equal(__isl_keep isl_set *set1, __isl_keep isl_set *set2);
__isl_export
isl_bool isl_set_is_disjoint(__isl_keep isl_set *set1,
	__isl_keep isl_set *set2);
__isl_export
isl_bool isl_set_is_singleton(__isl_keep isl_set *set);
isl_bool isl_set_is_box(__isl_keep isl_set *set);
isl_bool isl_set_has_equal_space(__isl_keep isl_set *set1,
	__isl_keep isl_set *set2);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_basic_set_involves_dims`, `isl_set_involves_dims`, `isl_set_print_internal`, `isl_set_plain_is_empty` (+11 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_basic_set_involves_dims`, `isl_set_involves_dims`, `isl_set_print_internal`, `isl_set_plain_is_empty` (+11 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 458-484

````cpp
__isl_give isl_set *isl_set_sum(__isl_take isl_set *set1,
	__isl_take isl_set *set2);
__isl_give isl_basic_set *isl_basic_set_neg(__isl_take isl_basic_set *bset);
__isl_give isl_set *isl_set_neg(__isl_take isl_set *set);

__isl_give isl_set *isl_set_make_disjoint(__isl_take isl_set *set);
__isl_give isl_set *isl_basic_set_compute_divs(__isl_take isl_basic_set *bset);
__isl_give isl_set *isl_set_compute_divs(__isl_take isl_set *set);
ISL_DEPRECATED
__isl_give isl_set *isl_set_align_divs(__isl_take isl_set *set);

__isl_export
__isl_give isl_multi_val *isl_set_get_plain_multi_val_if_fixed(
	__isl_keep isl_set *set);
__isl_give isl_val *isl_set_plain_get_val_if_fixed(__isl_keep isl_set *set,
	enum isl_dim_type type, unsigned pos);
isl_bool isl_set_dim_is_bounded(__isl_keep isl_set *set,
	enum isl_dim_type type, unsigned pos);
isl_bool isl_set_dim_has_lower_bound(__isl_keep isl_set *set,
	enum isl_dim_type type, unsigned pos);
isl_bool isl_set_dim_has_upper_bound(__isl_keep isl_set *set,
	enum isl_dim_type type, unsigned pos);
isl_bool isl_set_dim_has_any_lower_bound(__isl_keep isl_set *set,
	enum isl_dim_type type, unsigned pos);
isl_bool isl_set_dim_has_any_upper_bound(__isl_keep isl_set *set,
	enum isl_dim_type type, unsigned pos);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_set_sum`, `isl_basic_set_neg`, `isl_set_neg`, `isl_set_make_disjoint` (+10 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_set_sum`, `isl_basic_set_neg`, `isl_set_neg`, `isl_set_make_disjoint` (+10 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 485-508

````cpp
__isl_export
__isl_give isl_basic_set *isl_basic_set_gist(__isl_take isl_basic_set *bset,
					    __isl_take isl_basic_set *context);
__isl_give isl_set *isl_set_gist_basic_set(__isl_take isl_set *set,
	__isl_take isl_basic_set *context);
__isl_export
__isl_give isl_set *isl_set_gist(__isl_take isl_set *set,
	__isl_take isl_set *context);
__isl_export
__isl_give isl_set *isl_set_gist_params(__isl_take isl_set *set,
	__isl_take isl_set *context);
isl_stat isl_set_dim_residue_class_val(__isl_keep isl_set *set,
	int pos, __isl_give isl_val **modulo, __isl_give isl_val **residue);

__isl_give isl_stride_info *isl_set_get_stride_info(__isl_keep isl_set *set,
	int pos);
__isl_export
__isl_give isl_val *isl_set_get_stride(__isl_keep isl_set *set, int pos);
__isl_export
__isl_give isl_fixed_box *isl_set_get_lattice_tile(__isl_keep isl_set *set);
__isl_export
__isl_give isl_fixed_box *isl_set_get_simple_fixed_box_hull(
	__isl_keep isl_set *set);

````
- **EN**: This block declares or defines routines around `isl_basic_set_gist`, `isl_set_gist_basic_set`, `isl_set_gist`, `isl_set_gist_params` (+5 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_basic_set_gist`, `isl_set_gist_basic_set`, `isl_set_gist`, `isl_set_gist_params` (+5 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 509-532

````cpp
__isl_export
__isl_give isl_set *isl_set_coalesce(__isl_take isl_set *set);

int isl_set_plain_cmp(__isl_keep isl_set *set1, __isl_keep isl_set *set2);
isl_bool isl_set_plain_is_equal(__isl_keep isl_set *set1,
	__isl_keep isl_set *set2);
isl_bool isl_set_plain_is_disjoint(__isl_keep isl_set *set1,
	__isl_keep isl_set *set2);

uint32_t isl_set_get_hash(__isl_keep isl_set *set);

__isl_export
isl_size isl_set_n_basic_set(__isl_keep isl_set *set);
__isl_export
isl_stat isl_set_foreach_basic_set(__isl_keep isl_set *set,
	isl_stat (*fn)(__isl_take isl_basic_set *bset, void *user), void *user);
__isl_give isl_basic_set_list *isl_set_get_basic_set_list(
	__isl_keep isl_set *set);

__isl_export
isl_stat isl_set_foreach_point(__isl_keep isl_set *set,
	isl_stat (*fn)(__isl_take isl_point *pnt, void *user), void *user);
__isl_give isl_val *isl_set_count_val(__isl_keep isl_set *set);

````
- **EN**: This block declares or defines routines around `isl_set_coalesce`, `isl_set_plain_cmp`, `isl_set_plain_is_equal`, `isl_set_plain_is_disjoint` (+7 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_set_coalesce`, `isl_set_plain_cmp`, `isl_set_plain_is_equal`, `isl_set_plain_is_disjoint` (+7 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 533-555

````cpp
__isl_constructor
__isl_give isl_basic_set *isl_basic_set_from_point(__isl_take isl_point *pnt);
__isl_export
__isl_give isl_set *isl_point_to_set(__isl_take isl_point *pnt);
__isl_constructor
__isl_give isl_set *isl_set_from_point(__isl_take isl_point *pnt);
__isl_give isl_basic_set *isl_basic_set_box_from_points(
	__isl_take isl_point *pnt1, __isl_take isl_point *pnt2);
__isl_give isl_set *isl_set_box_from_points(__isl_take isl_point *pnt1,
	__isl_take isl_point *pnt2);

__isl_give isl_basic_set *isl_basic_set_lift(__isl_take isl_basic_set *bset);
__isl_give isl_set *isl_set_lift(__isl_take isl_set *set);

__isl_give isl_map *isl_set_lex_le_set(__isl_take isl_set *set1,
	__isl_take isl_set *set2);
__isl_give isl_map *isl_set_lex_lt_set(__isl_take isl_set *set1,
	__isl_take isl_set *set2);
__isl_give isl_map *isl_set_lex_ge_set(__isl_take isl_set *set1,
	__isl_take isl_set *set2);
__isl_give isl_map *isl_set_lex_gt_set(__isl_take isl_set *set1,
	__isl_take isl_set *set2);

````
- **EN**: This block declares or defines routines around `isl_basic_set_from_point`, `isl_point_to_set`, `isl_set_from_point`, `isl_basic_set_box_from_points` (+7 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_basic_set_from_point`, `isl_point_to_set`, `isl_set_from_point`, `isl_basic_set_box_from_points` (+7 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 556-577

````cpp
int isl_set_size(__isl_keep isl_set *set);

__isl_give isl_basic_set *isl_basic_set_align_params(
	__isl_take isl_basic_set *bset, __isl_take isl_space *model);
__isl_give isl_set *isl_set_align_params(__isl_take isl_set *set,
	__isl_take isl_space *model);
__isl_give isl_basic_set *isl_basic_set_drop_unused_params(
	__isl_take isl_basic_set *bset);
__isl_export
__isl_give isl_set *isl_set_drop_unused_params(__isl_take isl_set *set);

__isl_give isl_mat *isl_basic_set_equalities_matrix(
	__isl_keep isl_basic_set *bset, enum isl_dim_type c1,
	enum isl_dim_type c2, enum isl_dim_type c3, enum isl_dim_type c4);
__isl_give isl_mat *isl_basic_set_inequalities_matrix(
	__isl_keep isl_basic_set *bset, enum isl_dim_type c1,
	enum isl_dim_type c2, enum isl_dim_type c3, enum isl_dim_type c4);
__isl_give isl_basic_set *isl_basic_set_from_constraint_matrices(
	__isl_take isl_space *space,
	__isl_take isl_mat *eq, __isl_take isl_mat *ineq, enum isl_dim_type c1,
	enum isl_dim_type c2, enum isl_dim_type c3, enum isl_dim_type c4);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_set_size`, `isl_basic_set_align_params`, `isl_set_align_params`, `isl_basic_set_drop_unused_params` (+4 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_set_size`, `isl_basic_set_align_params`, `isl_set_align_params`, `isl_basic_set_drop_unused_params` (+4 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 578-598

````cpp
__isl_give isl_basic_set *isl_basic_set_from_multi_aff(
	__isl_take isl_multi_aff *ma);

__isl_export
__isl_give isl_set *isl_multi_aff_as_set(__isl_take isl_multi_aff *ma);
__isl_give isl_set *isl_set_from_multi_aff(__isl_take isl_multi_aff *ma);

__isl_give isl_mat *isl_basic_set_reduced_basis(__isl_keep isl_basic_set *bset);

__isl_give isl_basic_set *isl_basic_set_coefficients(
	__isl_take isl_basic_set *bset);
__isl_give isl_basic_set_list *isl_basic_set_list_coefficients(
	__isl_take isl_basic_set_list *list);
__isl_give isl_basic_set *isl_set_coefficients(__isl_take isl_set *set);
__isl_give isl_basic_set *isl_basic_set_solutions(
	__isl_take isl_basic_set *bset);
__isl_give isl_basic_set *isl_set_solutions(__isl_take isl_set *set);

__isl_give isl_pw_aff *isl_set_dim_max(__isl_take isl_set *set, int pos);
__isl_give isl_pw_aff *isl_set_dim_min(__isl_take isl_set *set, int pos);

````
- **EN**: This block declares or defines routines around `isl_basic_set_from_multi_aff`, `isl_multi_aff_as_set`, `isl_set_from_multi_aff`, `isl_basic_set_reduced_basis` (+7 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_basic_set_from_multi_aff`, `isl_multi_aff_as_set`, `isl_set_from_multi_aff`, `isl_basic_set_reduced_basis` (+7 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 599-606

````cpp
__isl_give char *isl_basic_set_to_str(__isl_keep isl_basic_set *bset);
__isl_give char *isl_set_to_str(__isl_keep isl_set *set);

#if defined(__cplusplus)
}
#endif

#endif
````
- **EN**: This block uses preprocessor directives to control compilation, macros, or include guards; declares or defines routines around `isl_basic_set_to_str`, `isl_set_to_str`; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 使用预处理指令控制编译、宏或头文件保护; 声明或定义与 `isl_basic_set_to_str`, `isl_set_to_str` 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

## Key Concepts / 关键概念

- **ISL relations and sets**
  - **CN**: ISL 关系与集合
- **Template/type wrappers**
  - **CN**: 模板与类型包装
- **Loop transformation**
  - **CN**: 循环变换
- **Public interface declarations**
  - **CN**: 公共接口声明

## Dependencies / 依赖关系

- **ISL headers**: `isl/id_type.h`, `isl/map_type.h`, `isl/aff_type.h`, `isl/list.h`, `isl/mat.h`, `isl/point.h`, `isl/local_space.h`, `isl/val_type.h` (+3 more) — ISL relation/set primitives that support polyhedral analysis and transformations.
  **ISL headers（CN）**：`isl/id_type.h`, `isl/map_type.h`, `isl/aff_type.h`, `isl/list.h`, `isl/mat.h`, `isl/point.h`, `isl/local_space.h`, `isl/val_type.h` (+3 more) —— 支撑多面体分析与变换的 ISL 关系/集合原语。
