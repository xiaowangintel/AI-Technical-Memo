# map.h — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `polly/lib/External/isl/include/isl/map.h` | `polly/lib/External/isl/include/isl/map.h` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | Declares upstream ISL interfaces and C++ wrappers that Polly depends on for polyhedral operations. | 声明 Polly 依赖的上游 ISL 接口与 C++ 包装层，用于多面体计算。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-28

````cpp
/*
 * Copyright 2008-2009 Katholieke Universiteit Leuven
 *
 * Use of this software is governed by the MIT license
 *
 * Written by Sven Verdoolaege, K.U.Leuven, Departement
 * Computerwetenschappen, Celestijnenlaan 200A, B-3001 Leuven, Belgium
 */

#ifndef ISL_MAP_H
#define ISL_MAP_H

#include <stdio.h>

#include <isl/ctx.h>
#include <isl/space_type.h>
#include <isl/vec.h>
#include <isl/mat.h>
#include <isl/printer.h>
#include <isl/local_space.h>
#include <isl/aff_type.h>
#include <isl/list.h>
#include <isl/map_type.h>
#include <isl/val_type.h>
#include <isl/stdint.h>
#include <isl/stride_info.h>
#include <isl/fixed_box.h>

````
- **EN**: This block imports ISL, system/standard headers needed by the surrounding code; uses preprocessor directives to control compilation, macros, or include guards; defines macros like `ISL_MAP_H`; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 引入周边逻辑所需的 ISL、system/standard 头文件; 使用预处理指令控制编译、宏或头文件保护; 定义宏，例如 `ISL_MAP_H`; 保留用于解释意图、用法或算法背景的注释.

### Lines 29-48

````cpp
#if defined(__cplusplus)
extern "C" {
#endif

isl_size isl_basic_map_total_dim(__isl_keep const isl_basic_map *bmap);
isl_size isl_basic_map_dim(__isl_keep isl_basic_map *bmap,
				enum isl_dim_type type);

__isl_export
isl_size isl_map_domain_tuple_dim(__isl_keep isl_map *map);
__isl_export
isl_size isl_map_range_tuple_dim(__isl_keep isl_map *map);
isl_size isl_map_dim(__isl_keep isl_map *map, enum isl_dim_type type);

isl_ctx *isl_basic_map_get_ctx(__isl_keep isl_basic_map *bmap);
isl_ctx *isl_map_get_ctx(__isl_keep isl_map *map);
__isl_give isl_space *isl_basic_map_get_space(__isl_keep isl_basic_map *bmap);
__isl_export
__isl_give isl_space *isl_map_get_space(__isl_keep isl_map *map);

````
- **EN**: This block uses preprocessor directives to control compilation, macros, or include guards; defines enum values such as `isl_dim_type`; declares or defines routines around `isl_basic_map_total_dim`, `isl_basic_map_dim`, `isl_map_domain_tuple_dim`, `isl_map_range_tuple_dim` (+5 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 使用预处理指令控制编译、宏或头文件保护; 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_basic_map_total_dim`, `isl_basic_map_dim`, `isl_map_domain_tuple_dim`, `isl_map_range_tuple_dim` (+5 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 49-76

````cpp
__isl_give isl_aff *isl_basic_map_get_div(__isl_keep isl_basic_map *bmap,
	int pos);

__isl_give isl_local_space *isl_basic_map_get_local_space(
	__isl_keep isl_basic_map *bmap);

__isl_give isl_basic_map *isl_basic_map_set_tuple_name(
	__isl_take isl_basic_map *bmap, enum isl_dim_type type, const char *s);
const char *isl_basic_map_get_tuple_name(__isl_keep isl_basic_map *bmap,
	enum isl_dim_type type);
isl_bool isl_map_has_tuple_name(__isl_keep isl_map *map,
	enum isl_dim_type type);
const char *isl_map_get_tuple_name(__isl_keep isl_map *map,
	enum isl_dim_type type);
__isl_give isl_map *isl_map_set_tuple_name(__isl_take isl_map *map,
	enum isl_dim_type type, const char *s);
const char *isl_basic_map_get_dim_name(__isl_keep isl_basic_map *bmap,
	enum isl_dim_type type, unsigned pos);
isl_bool isl_map_has_dim_name(__isl_keep isl_map *map,
	enum isl_dim_type type, unsigned pos);
const char *isl_map_get_dim_name(__isl_keep isl_map *map,
	enum isl_dim_type type, unsigned pos);
__isl_give isl_basic_map *isl_basic_map_set_dim_name(
	__isl_take isl_basic_map *bmap,
	enum isl_dim_type type, unsigned pos, const char *s);
__isl_give isl_map *isl_map_set_dim_name(__isl_take isl_map *map,
	enum isl_dim_type type, unsigned pos, const char *s);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_basic_map_get_div`, `isl_basic_map_get_local_space`, `isl_basic_map_set_tuple_name`, `isl_basic_map_get_tuple_name` (+8 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_basic_map_get_div`, `isl_basic_map_get_local_space`, `isl_basic_map_set_tuple_name`, `isl_basic_map_get_tuple_name` (+8 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 77-100

````cpp
__isl_give isl_basic_map *isl_basic_map_set_tuple_id(
	__isl_take isl_basic_map *bmap,
	enum isl_dim_type type, __isl_take isl_id *id);
__isl_give isl_map *isl_map_set_dim_id(__isl_take isl_map *map,
	enum isl_dim_type type, unsigned pos, __isl_take isl_id *id);
isl_bool isl_basic_map_has_dim_id(__isl_keep isl_basic_map *bmap,
	enum isl_dim_type type, unsigned pos);
isl_bool isl_map_has_dim_id(__isl_keep isl_map *map,
	enum isl_dim_type type, unsigned pos);
__isl_give isl_id *isl_map_get_dim_id(__isl_keep isl_map *map,
	enum isl_dim_type type, unsigned pos);
__isl_overload
__isl_give isl_map *isl_map_set_domain_tuple_id(__isl_take isl_map *map,
	__isl_take isl_id *id);
__isl_overload
__isl_give isl_map *isl_map_set_range_tuple_id(__isl_take isl_map *map,
	__isl_take isl_id *id);
__isl_give isl_map *isl_map_set_tuple_id(__isl_take isl_map *map,
	enum isl_dim_type type, __isl_take isl_id *id);
__isl_give isl_map *isl_map_reset_tuple_id(__isl_take isl_map *map,
	enum isl_dim_type type);
__isl_export
isl_bool isl_map_has_domain_tuple_id(__isl_keep isl_map *map);
__isl_export
````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_basic_map_set_tuple_id`, `isl_map_set_dim_id`, `isl_basic_map_has_dim_id`, `isl_map_has_dim_id` (+6 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_basic_map_set_tuple_id`, `isl_map_set_dim_id`, `isl_basic_map_has_dim_id`, `isl_map_has_dim_id` (+6 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 101-124

````cpp
isl_bool isl_map_has_range_tuple_id(__isl_keep isl_map *map);
isl_bool isl_map_has_tuple_id(__isl_keep isl_map *map, enum isl_dim_type type);
__isl_export
__isl_give isl_id *isl_map_get_domain_tuple_id(__isl_keep isl_map *map);
__isl_export
__isl_give isl_id *isl_map_get_range_tuple_id(__isl_keep isl_map *map);
__isl_give isl_id *isl_map_get_tuple_id(__isl_keep isl_map *map,
	enum isl_dim_type type);
__isl_give isl_map *isl_map_reset_user(__isl_take isl_map *map);

int isl_basic_map_find_dim_by_name(__isl_keep isl_basic_map *bmap,
	enum isl_dim_type type, const char *name);
int isl_map_find_dim_by_id(__isl_keep isl_map *map, enum isl_dim_type type,
	__isl_keep isl_id *id);
int isl_map_find_dim_by_name(__isl_keep isl_map *map, enum isl_dim_type type,
	const char *name);

isl_bool isl_basic_map_is_rational(__isl_keep isl_basic_map *bmap);

__isl_give isl_basic_map *isl_basic_map_identity(__isl_take isl_space *space);
__isl_null isl_basic_map *isl_basic_map_free(__isl_take isl_basic_map *bmap);
__isl_give isl_basic_map *isl_basic_map_copy(__isl_keep isl_basic_map *bmap);
__isl_give isl_basic_map *isl_basic_map_equal(
	__isl_take isl_space *space, unsigned n_equal);
````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_map_has_range_tuple_id`, `isl_map_has_tuple_id`, `isl_map_get_domain_tuple_id`, `isl_map_get_range_tuple_id` (+10 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_map_has_range_tuple_id`, `isl_map_has_tuple_id`, `isl_map_get_domain_tuple_id`, `isl_map_get_range_tuple_id` (+10 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 125-144

````cpp
__isl_give isl_basic_map *isl_basic_map_less_at(__isl_take isl_space *space,
	unsigned pos);
__isl_give isl_basic_map *isl_basic_map_more_at(__isl_take isl_space *space,
	unsigned pos);
__isl_give isl_basic_map *isl_basic_map_empty(__isl_take isl_space *space);
__isl_give isl_basic_map *isl_basic_map_universe(__isl_take isl_space *space);
__isl_give isl_basic_map *isl_basic_map_nat_universe(
	__isl_take isl_space *space);
__isl_give isl_basic_map *isl_basic_map_remove_redundancies(
	__isl_take isl_basic_map *bmap);
__isl_give isl_map *isl_map_remove_redundancies(__isl_take isl_map *map);
__isl_give isl_basic_map *isl_map_simple_hull(__isl_take isl_map *map);
__isl_export
__isl_give isl_basic_map *isl_map_unshifted_simple_hull(
	__isl_take isl_map *map);
__isl_give isl_basic_map *isl_map_plain_unshifted_simple_hull(
	__isl_take isl_map *map);
__isl_give isl_basic_map *isl_map_unshifted_simple_hull_from_map_list(
	__isl_take isl_map *map, __isl_take isl_map_list *list);

````
- **EN**: This block declares or defines routines around `isl_basic_map_less_at`, `isl_basic_map_more_at`, `isl_basic_map_empty`, `isl_basic_map_universe` (+7 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_basic_map_less_at`, `isl_basic_map_more_at`, `isl_basic_map_empty`, `isl_basic_map_universe` (+7 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 145-168

````cpp
__isl_export
__isl_give isl_basic_map *isl_basic_map_intersect_params(
	__isl_take isl_basic_map *bmap, __isl_take isl_basic_set *bset);
__isl_export
__isl_give isl_basic_map *isl_basic_map_intersect_domain(
		__isl_take isl_basic_map *bmap,
		__isl_take isl_basic_set *bset);
__isl_export
__isl_give isl_basic_map *isl_basic_map_intersect_range(
		__isl_take isl_basic_map *bmap,
		__isl_take isl_basic_set *bset);
__isl_export
__isl_give isl_basic_map *isl_basic_map_intersect(
		__isl_take isl_basic_map *bmap1,
		__isl_take isl_basic_map *bmap2);
__isl_give isl_basic_map *isl_basic_map_list_intersect(
	__isl_take isl_basic_map_list *list);
__isl_export
__isl_give isl_map *isl_basic_map_union(
		__isl_take isl_basic_map *bmap1,
		__isl_take isl_basic_map *bmap2);
__isl_export
__isl_give isl_basic_map *isl_basic_map_apply_domain(
		__isl_take isl_basic_map *bmap1,
````
- **EN**: This block declares or defines routines around `isl_basic_map_intersect_params`, `isl_basic_map_intersect_domain`, `isl_basic_map_intersect_range`, `isl_basic_map_intersect` (+3 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_basic_map_intersect_params`, `isl_basic_map_intersect_domain`, `isl_basic_map_intersect_range`, `isl_basic_map_intersect` (+3 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 169-192

````cpp
		__isl_take isl_basic_map *bmap2);
__isl_export
__isl_give isl_basic_map *isl_basic_map_apply_range(
		__isl_take isl_basic_map *bmap1,
		__isl_take isl_basic_map *bmap2);
__isl_export
__isl_give isl_basic_map *isl_basic_map_affine_hull(
		__isl_take isl_basic_map *bmap);
__isl_give isl_basic_map *isl_basic_map_preimage_domain_multi_aff(
	__isl_take isl_basic_map *bmap, __isl_take isl_multi_aff *ma);
__isl_give isl_basic_map *isl_basic_map_preimage_range_multi_aff(
	__isl_take isl_basic_map *bmap, __isl_take isl_multi_aff *ma);
__isl_export
__isl_give isl_basic_map *isl_basic_map_reverse(__isl_take isl_basic_map *bmap);
__isl_give isl_basic_set *isl_basic_map_domain(__isl_take isl_basic_map *bmap);
__isl_give isl_basic_set *isl_basic_map_range(__isl_take isl_basic_map *bmap);
__isl_give isl_basic_map *isl_basic_map_domain_map(
	__isl_take isl_basic_map *bmap);
__isl_give isl_basic_map *isl_basic_map_range_map(
	__isl_take isl_basic_map *bmap);
__isl_give isl_basic_map *isl_basic_map_remove_dims(
	__isl_take isl_basic_map *bmap,
	enum isl_dim_type type, unsigned first, unsigned n);
__isl_give isl_basic_map *isl_basic_map_eliminate(
````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_basic_map_apply_range`, `isl_basic_map_affine_hull`, `isl_basic_map_preimage_domain_multi_aff`, `isl_basic_map_preimage_range_multi_aff` (+7 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_basic_map_apply_range`, `isl_basic_map_affine_hull`, `isl_basic_map_preimage_domain_multi_aff`, `isl_basic_map_preimage_range_multi_aff` (+7 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 193-216

````cpp
	__isl_take isl_basic_map *bmap,
	enum isl_dim_type type, unsigned first, unsigned n);
__isl_export
__isl_give isl_basic_map *isl_basic_map_sample(__isl_take isl_basic_map *bmap);
__isl_export
__isl_give isl_basic_map *isl_basic_map_detect_equalities(
						__isl_take isl_basic_map *bmap);
__isl_give isl_basic_map *isl_basic_map_read_from_file(isl_ctx *ctx,
	FILE *input);
__isl_constructor
__isl_give isl_basic_map *isl_basic_map_read_from_str(isl_ctx *ctx,
	const char *str);
__isl_give isl_map *isl_map_read_from_file(isl_ctx *ctx, FILE *input);
__isl_constructor
__isl_give isl_map *isl_map_read_from_str(isl_ctx *ctx, const char *str);
void isl_basic_map_dump(__isl_keep isl_basic_map *bmap);
void isl_map_dump(__isl_keep isl_map *map);
__isl_give char *isl_basic_map_to_str(__isl_keep isl_basic_map *bmap);
__isl_give isl_printer *isl_printer_print_basic_map(
	__isl_take isl_printer *printer, __isl_keep isl_basic_map *bmap);
__isl_give char *isl_map_to_str(__isl_keep isl_map *map);
__isl_give isl_printer *isl_printer_print_map(__isl_take isl_printer *printer,
	__isl_keep isl_map *map);
__isl_give isl_basic_map *isl_basic_map_fix_si(__isl_take isl_basic_map *bmap,
````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_basic_map_sample`, `isl_basic_map_detect_equalities`, `isl_basic_map_read_from_file`, `isl_basic_map_read_from_str` (+9 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_basic_map_sample`, `isl_basic_map_detect_equalities`, `isl_basic_map_read_from_file`, `isl_basic_map_read_from_str` (+9 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 217-236

````cpp
		enum isl_dim_type type, unsigned pos, int value);
__isl_give isl_basic_map *isl_basic_map_fix_val(__isl_take isl_basic_map *bmap,
	enum isl_dim_type type, unsigned pos, __isl_take isl_val *v);
__isl_give isl_basic_map *isl_basic_map_lower_bound_si(
		__isl_take isl_basic_map *bmap,
		enum isl_dim_type type, unsigned pos, int value);
__isl_give isl_basic_map *isl_basic_map_upper_bound_si(
	__isl_take isl_basic_map *bmap,
	enum isl_dim_type type, unsigned pos, int value);
__isl_overload
__isl_give isl_map *isl_map_lower_bound_multi_pw_aff(__isl_take isl_map *map,
	__isl_take isl_multi_pw_aff *lower);
__isl_overload
__isl_give isl_map *isl_map_upper_bound_multi_pw_aff(__isl_take isl_map *map,
	__isl_take isl_multi_pw_aff *upper);

__isl_give isl_basic_map *isl_basic_map_sum(__isl_take isl_basic_map *bmap1,
	__isl_take isl_basic_map *bmap2);
__isl_give isl_basic_map *isl_basic_map_neg(__isl_take isl_basic_map *bmap);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_basic_map_fix_val`, `isl_basic_map_lower_bound_si`, `isl_basic_map_upper_bound_si`, `isl_map_lower_bound_multi_pw_aff` (+3 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_basic_map_fix_val`, `isl_basic_map_lower_bound_si`, `isl_basic_map_upper_bound_si`, `isl_map_lower_bound_multi_pw_aff` (+3 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 237-260

````cpp
__isl_give isl_map *isl_map_sum(__isl_take isl_map *map1,
	__isl_take isl_map *map2);
__isl_give isl_map *isl_map_neg(__isl_take isl_map *map);
__isl_give isl_map *isl_map_floordiv_val(__isl_take isl_map *map,
	__isl_take isl_val *d);

__isl_export
isl_bool isl_basic_map_is_equal(__isl_keep isl_basic_map *bmap1,
	__isl_keep isl_basic_map *bmap2);
isl_bool isl_basic_map_is_disjoint(__isl_keep isl_basic_map *bmap1,
	__isl_keep isl_basic_map *bmap2);

__isl_give isl_map *isl_basic_map_partial_lexmax(
		__isl_take isl_basic_map *bmap, __isl_take isl_basic_set *dom,
		__isl_give isl_set **empty);
__isl_give isl_map *isl_basic_map_partial_lexmin(
		__isl_take isl_basic_map *bmap, __isl_take isl_basic_set *dom,
		__isl_give isl_set **empty);
__isl_give isl_map *isl_map_partial_lexmax(
		__isl_take isl_map *map, __isl_take isl_set *dom,
		__isl_give isl_set **empty);
__isl_give isl_map *isl_map_partial_lexmin(
		__isl_take isl_map *map, __isl_take isl_set *dom,
		__isl_give isl_set **empty);
````
- **EN**: This block declares or defines routines around `isl_map_sum`, `isl_map_neg`, `isl_map_floordiv_val`, `isl_basic_map_is_equal` (+5 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_map_sum`, `isl_map_neg`, `isl_map_floordiv_val`, `isl_basic_map_is_equal` (+5 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 261-287

````cpp
__isl_export
__isl_give isl_map *isl_basic_map_lexmin(__isl_take isl_basic_map *bmap);
__isl_export
__isl_give isl_map *isl_basic_map_lexmax(__isl_take isl_basic_map *bmap);
__isl_export
__isl_give isl_map *isl_map_lexmin(__isl_take isl_map *map);
__isl_export
__isl_give isl_map *isl_map_lexmax(__isl_take isl_map *map);
__isl_give isl_pw_multi_aff *isl_basic_map_partial_lexmin_pw_multi_aff(
	__isl_take isl_basic_map *bmap, __isl_take isl_basic_set *dom,
	__isl_give isl_set **empty);
__isl_give isl_pw_multi_aff *isl_basic_map_partial_lexmax_pw_multi_aff(
	__isl_take isl_basic_map *bmap, __isl_take isl_basic_set *dom,
	__isl_give isl_set **empty);
__isl_give isl_pw_multi_aff *isl_basic_map_lexmin_pw_multi_aff(
	__isl_take isl_basic_map *bmap);
__isl_export
__isl_give isl_pw_multi_aff *isl_map_lexmin_pw_multi_aff(
	__isl_take isl_map *map);
__isl_export
__isl_give isl_pw_multi_aff *isl_map_lexmax_pw_multi_aff(
	__isl_take isl_map *map);
__isl_export
__isl_give isl_multi_pw_aff *isl_map_min_multi_pw_aff(__isl_take isl_map *map);
__isl_export
__isl_give isl_multi_pw_aff *isl_map_max_multi_pw_aff(__isl_take isl_map *map);

````
- **EN**: This block declares or defines routines around `isl_basic_map_lexmin`, `isl_basic_map_lexmax`, `isl_map_lexmin`, `isl_map_lexmax` (+7 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_basic_map_lexmin`, `isl_basic_map_lexmax`, `isl_map_lexmin`, `isl_map_lexmax` (+7 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 288-311

````cpp
void isl_basic_map_print_internal(__isl_keep isl_basic_map *bmap,
	FILE *out, int indent);

__isl_give isl_val *isl_basic_map_plain_get_val_if_fixed(
	__isl_keep isl_basic_map *bmap,
	enum isl_dim_type type, unsigned pos);

isl_bool isl_basic_map_image_is_bounded(__isl_keep isl_basic_map *bmap);
isl_bool isl_basic_map_plain_is_universe(__isl_keep isl_basic_map *bmap);
isl_bool isl_basic_map_is_universe(__isl_keep isl_basic_map *bmap);
isl_bool isl_basic_map_plain_is_empty(__isl_keep isl_basic_map *bmap);
__isl_export
isl_bool isl_basic_map_is_empty(__isl_keep isl_basic_map *bmap);
__isl_export
isl_bool isl_basic_map_is_subset(__isl_keep isl_basic_map *bmap1,
		__isl_keep isl_basic_map *bmap2);
isl_bool isl_basic_map_is_strict_subset(__isl_keep isl_basic_map *bmap1,
		__isl_keep isl_basic_map *bmap2);

__isl_export
__isl_give isl_map *isl_map_universe(__isl_take isl_space *space);
__isl_export
__isl_give isl_map *isl_space_universe_map(__isl_take isl_space *space);
__isl_give isl_map *isl_map_nat_universe(__isl_take isl_space *space);
````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_basic_map_print_internal`, `isl_basic_map_plain_get_val_if_fixed`, `isl_basic_map_image_is_bounded`, `isl_basic_map_plain_is_universe` (+8 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_basic_map_print_internal`, `isl_basic_map_plain_get_val_if_fixed`, `isl_basic_map_image_is_bounded`, `isl_basic_map_plain_is_universe` (+8 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 312-335

````cpp
__isl_export
__isl_give isl_map *isl_map_empty(__isl_take isl_space *space);
__isl_give isl_map *isl_map_identity(__isl_take isl_space *space);
__isl_give isl_map *isl_map_lex_lt_first(__isl_take isl_space *space,
	unsigned n);
__isl_give isl_map *isl_map_lex_le_first(__isl_take isl_space *space,
	unsigned n);
__isl_give isl_map *isl_map_lex_lt(__isl_take isl_space *set_space);
__isl_give isl_map *isl_map_lex_le(__isl_take isl_space *set_space);
__isl_give isl_map *isl_map_lex_gt_first(__isl_take isl_space *space,
	unsigned n);
__isl_give isl_map *isl_map_lex_ge_first(__isl_take isl_space *space,
	unsigned n);
__isl_give isl_map *isl_map_lex_gt(__isl_take isl_space *set_space);
__isl_give isl_map *isl_map_lex_ge(__isl_take isl_space *set_space);
__isl_null isl_map *isl_map_free(__isl_take isl_map *map);
__isl_give isl_map *isl_map_copy(__isl_keep isl_map *map);
__isl_export
__isl_give isl_map *isl_map_reverse(__isl_take isl_map *map);
__isl_export
__isl_give isl_map *isl_map_domain_reverse(__isl_take isl_map *map);
__isl_export
__isl_give isl_map *isl_map_range_reverse(__isl_take isl_map *map);
__isl_export
````
- **EN**: This block declares or defines routines around `isl_map_empty`, `isl_map_identity`, `isl_map_lex_lt_first`, `isl_map_lex_le_first` (+11 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_map_empty`, `isl_map_identity`, `isl_map_lex_lt_first`, `isl_map_lex_le_first` (+11 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 336-359

````cpp
__isl_give isl_map *isl_map_union(
		__isl_take isl_map *map1,
		__isl_take isl_map *map2);
__isl_give isl_map *isl_map_union_disjoint(
		__isl_take isl_map *map1, __isl_take isl_map *map2);
__isl_export
__isl_give isl_map *isl_map_intersect_domain(
		__isl_take isl_map *map,
		__isl_take isl_set *set);
__isl_export
__isl_give isl_map *isl_map_intersect_range(
		__isl_take isl_map *map,
		__isl_take isl_set *set);
__isl_export
__isl_give isl_map *isl_map_intersect_domain_factor_domain(
	__isl_take isl_map *map, __isl_take isl_map *factor);
__isl_export
__isl_give isl_map *isl_map_intersect_domain_factor_range(
	__isl_take isl_map *map, __isl_take isl_map *factor);
__isl_export
__isl_give isl_map *isl_map_intersect_range_factor_domain(
	__isl_take isl_map *map, __isl_take isl_map *factor);
__isl_export
__isl_give isl_map *isl_map_intersect_range_factor_range(
````
- **EN**: This block declares or defines routines around `isl_map_union`, `isl_map_union_disjoint`, `isl_map_intersect_domain`, `isl_map_intersect_range` (+4 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_map_union`, `isl_map_union_disjoint`, `isl_map_intersect_domain`, `isl_map_intersect_range` (+4 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 360-383

````cpp
	__isl_take isl_map *map, __isl_take isl_map *factor);
__isl_export
__isl_give isl_map *isl_map_intersect_domain_wrapped_domain(
	__isl_take isl_map *map, __isl_take isl_set *domain);
__isl_export
__isl_give isl_map *isl_map_intersect_range_wrapped_domain(
	__isl_take isl_map *map, __isl_take isl_set *domain);
__isl_export
__isl_give isl_map *isl_map_apply_domain(
		__isl_take isl_map *map1,
		__isl_take isl_map *map2);
__isl_export
__isl_give isl_map *isl_map_apply_range(
		__isl_take isl_map *map1,
		__isl_take isl_map *map2);
__isl_overload
__isl_give isl_map *isl_map_preimage_domain_multi_aff(__isl_take isl_map *map,
	__isl_take isl_multi_aff *ma);
__isl_overload
__isl_give isl_map *isl_map_preimage_range_multi_aff(__isl_take isl_map *map,
	__isl_take isl_multi_aff *ma);
__isl_overload
__isl_give isl_map *isl_map_preimage_domain_pw_multi_aff(
	__isl_take isl_map *map, __isl_take isl_pw_multi_aff *pma);
````
- **EN**: This block declares or defines routines around `isl_map_intersect_domain_wrapped_domain`, `isl_map_intersect_range_wrapped_domain`, `isl_map_apply_domain`, `isl_map_apply_range` (+3 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_map_intersect_domain_wrapped_domain`, `isl_map_intersect_range_wrapped_domain`, `isl_map_apply_domain`, `isl_map_apply_range` (+3 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 384-407

````cpp
__isl_overload
__isl_give isl_map *isl_map_preimage_range_pw_multi_aff(
	__isl_take isl_map *map, __isl_take isl_pw_multi_aff *pma);
__isl_overload
__isl_give isl_map *isl_map_preimage_domain_multi_pw_aff(
	__isl_take isl_map *map, __isl_take isl_multi_pw_aff *mpa);
__isl_give isl_basic_map *isl_basic_map_product(
	__isl_take isl_basic_map *bmap1, __isl_take isl_basic_map *bmap2);
__isl_export
__isl_give isl_map *isl_map_product(__isl_take isl_map *map1,
	__isl_take isl_map *map2);
__isl_give isl_basic_map *isl_basic_map_domain_product(
	__isl_take isl_basic_map *bmap1, __isl_take isl_basic_map *bmap2);
__isl_give isl_basic_map *isl_basic_map_range_product(
	__isl_take isl_basic_map *bmap1, __isl_take isl_basic_map *bmap2);
__isl_export
__isl_give isl_map *isl_map_domain_product(__isl_take isl_map *map1,
	__isl_take isl_map *map2);
__isl_export
__isl_give isl_map *isl_map_range_product(__isl_take isl_map *map1,
	__isl_take isl_map *map2);
__isl_give isl_basic_map *isl_basic_map_flat_product(
	__isl_take isl_basic_map *bmap1, __isl_take isl_basic_map *bmap2);
__isl_give isl_map *isl_map_flat_product(__isl_take isl_map *map1,
````
- **EN**: This block declares or defines routines around `isl_map_preimage_range_pw_multi_aff`, `isl_map_preimage_domain_multi_pw_aff`, `isl_basic_map_product`, `isl_map_product` (+6 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_map_preimage_range_pw_multi_aff`, `isl_map_preimage_domain_multi_pw_aff`, `isl_basic_map_product`, `isl_map_product` (+6 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 408-431

````cpp
	__isl_take isl_map *map2);
__isl_give isl_basic_map *isl_basic_map_flat_range_product(
	__isl_take isl_basic_map *bmap1, __isl_take isl_basic_map *bmap2);
__isl_give isl_map *isl_map_flat_domain_product(__isl_take isl_map *map1,
	__isl_take isl_map *map2);
__isl_give isl_map *isl_map_flat_range_product(__isl_take isl_map *map1,
	__isl_take isl_map *map2);
isl_bool isl_map_domain_is_wrapping(__isl_keep isl_map *map);
isl_bool isl_map_range_is_wrapping(__isl_keep isl_map *map);
isl_bool isl_map_is_product(__isl_keep isl_map *map);
__isl_export
__isl_give isl_map *isl_map_factor_domain(__isl_take isl_map *map);
__isl_export
__isl_give isl_map *isl_map_factor_range(__isl_take isl_map *map);
__isl_export
__isl_give isl_map *isl_map_domain_factor_domain(__isl_take isl_map *map);
__isl_export
__isl_give isl_map *isl_map_domain_factor_range(__isl_take isl_map *map);
__isl_export
__isl_give isl_map *isl_map_range_factor_domain(__isl_take isl_map *map);
__isl_export
__isl_give isl_map *isl_map_range_factor_range(__isl_take isl_map *map);
__isl_export
__isl_give isl_map *isl_map_intersect(__isl_take isl_map *map1,
````
- **EN**: This block declares or defines routines around `isl_basic_map_flat_range_product`, `isl_map_flat_domain_product`, `isl_map_flat_range_product`, `isl_map_domain_is_wrapping` (+9 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_basic_map_flat_range_product`, `isl_map_flat_domain_product`, `isl_map_flat_range_product`, `isl_map_domain_is_wrapping` (+9 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 432-455

````cpp
				      __isl_take isl_map *map2);
__isl_export
__isl_give isl_map *isl_map_intersect_params(__isl_take isl_map *map,
		__isl_take isl_set *params);
__isl_export
__isl_give isl_map *isl_map_subtract(
		__isl_take isl_map *map1,
		__isl_take isl_map *map2);
__isl_give isl_map *isl_map_subtract_domain(__isl_take isl_map *map,
	__isl_take isl_set *dom);
__isl_give isl_map *isl_map_subtract_range(__isl_take isl_map *map,
	__isl_take isl_set *dom);
__isl_export
__isl_give isl_map *isl_map_complement(__isl_take isl_map *map);
__isl_give isl_map *isl_map_fix_input_si(__isl_take isl_map *map,
		unsigned input, int value);
__isl_give isl_map *isl_map_fix_si(__isl_take isl_map *map,
		enum isl_dim_type type, unsigned pos, int value);
__isl_give isl_map *isl_map_fix_val(__isl_take isl_map *map,
	enum isl_dim_type type, unsigned pos, __isl_take isl_val *v);
__isl_give isl_map *isl_map_lower_bound_si(__isl_take isl_map *map,
		enum isl_dim_type type, unsigned pos, int value);
__isl_give isl_map *isl_map_lower_bound_val(__isl_take isl_map *map,
	enum isl_dim_type type, unsigned pos, __isl_take isl_val *value);
````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_map_intersect_params`, `isl_map_subtract`, `isl_map_subtract_domain`, `isl_map_subtract_range` (+6 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_map_intersect_params`, `isl_map_subtract`, `isl_map_subtract_domain`, `isl_map_subtract_range` (+6 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 456-479

````cpp
__isl_give isl_map *isl_map_upper_bound_si(__isl_take isl_map *map,
	enum isl_dim_type type, unsigned pos, int value);
__isl_give isl_map *isl_map_upper_bound_val(__isl_take isl_map *map,
	enum isl_dim_type type, unsigned pos, __isl_take isl_val *value);
__isl_export
__isl_give isl_basic_set *isl_basic_map_deltas(__isl_take isl_basic_map *bmap);
__isl_export
__isl_give isl_set *isl_map_deltas(__isl_take isl_map *map);
__isl_give isl_basic_map *isl_basic_map_deltas_map(
	__isl_take isl_basic_map *bmap);
__isl_give isl_map *isl_map_deltas_map(__isl_take isl_map *map);
__isl_export
__isl_give isl_map *isl_map_detect_equalities(__isl_take isl_map *map);
__isl_export
__isl_give isl_basic_map *isl_map_affine_hull(__isl_take isl_map *map);
__isl_give isl_basic_map *isl_map_convex_hull(__isl_take isl_map *map);
__isl_export
__isl_give isl_basic_map *isl_map_polyhedral_hull(__isl_take isl_map *map);
__isl_give isl_basic_map *isl_basic_map_add_dims(__isl_take isl_basic_map *bmap,
		enum isl_dim_type type, unsigned n);
__isl_give isl_map *isl_map_add_dims(__isl_take isl_map *map,
		enum isl_dim_type type, unsigned n);
__isl_give isl_basic_map *isl_basic_map_insert_dims(
	__isl_take isl_basic_map *bmap, enum isl_dim_type type,
````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_map_upper_bound_si`, `isl_map_upper_bound_val`, `isl_basic_map_deltas`, `isl_map_deltas` (+9 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_map_upper_bound_si`, `isl_map_upper_bound_val`, `isl_basic_map_deltas`, `isl_map_deltas` (+9 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 480-503

````cpp
	unsigned pos, unsigned n);
__isl_give isl_map *isl_map_insert_dims(__isl_take isl_map *map,
		enum isl_dim_type type, unsigned pos, unsigned n);
__isl_give isl_basic_map *isl_basic_map_move_dims(
	__isl_take isl_basic_map *bmap,
	enum isl_dim_type dst_type, unsigned dst_pos,
	enum isl_dim_type src_type, unsigned src_pos, unsigned n);
__isl_give isl_map *isl_map_move_dims(__isl_take isl_map *map,
	enum isl_dim_type dst_type, unsigned dst_pos,
	enum isl_dim_type src_type, unsigned src_pos, unsigned n);
__isl_give isl_basic_map *isl_basic_map_project_out(
		__isl_take isl_basic_map *bmap,
		enum isl_dim_type type, unsigned first, unsigned n);
__isl_overload
__isl_give isl_map *isl_map_project_out_param_id(__isl_take isl_map *map,
	__isl_take isl_id *id);
__isl_overload
__isl_give isl_map *isl_map_project_out_param_id_list(__isl_take isl_map *map,
	__isl_take isl_id_list *list);
__isl_give isl_map *isl_map_project_out(__isl_take isl_map *map,
		enum isl_dim_type type, unsigned first, unsigned n);
__isl_export
__isl_give isl_map *isl_map_project_out_all_params(__isl_take isl_map *map);
__isl_give isl_basic_map *isl_basic_map_remove_divs(
````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_map_insert_dims`, `isl_basic_map_move_dims`, `isl_map_move_dims`, `isl_basic_map_project_out` (+5 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_map_insert_dims`, `isl_basic_map_move_dims`, `isl_map_move_dims`, `isl_basic_map_project_out` (+5 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 504-527

````cpp
	__isl_take isl_basic_map *bmap);
__isl_give isl_map *isl_map_remove_unknown_divs(__isl_take isl_map *map);
__isl_give isl_map *isl_map_remove_divs(__isl_take isl_map *map);
__isl_give isl_map *isl_map_eliminate(__isl_take isl_map *map,
	enum isl_dim_type type, unsigned first, unsigned n);
__isl_give isl_map *isl_map_remove_dims(__isl_take isl_map *map,
	enum isl_dim_type type, unsigned first, unsigned n);
__isl_give isl_basic_map *isl_basic_map_remove_divs_involving_dims(
	__isl_take isl_basic_map *bmap,
	enum isl_dim_type type, unsigned first, unsigned n);
__isl_give isl_map *isl_map_remove_divs_involving_dims(__isl_take isl_map *map,
	enum isl_dim_type type, unsigned first, unsigned n);
__isl_give isl_map *isl_map_remove_inputs(__isl_take isl_map *map,
	unsigned first, unsigned n);

__isl_give isl_basic_map *isl_basic_map_equate(__isl_take isl_basic_map *bmap,
	enum isl_dim_type type1, int pos1, enum isl_dim_type type2, int pos2);
__isl_give isl_basic_map *isl_basic_map_order_ge(__isl_take isl_basic_map *bmap,
	enum isl_dim_type type1, int pos1, enum isl_dim_type type2, int pos2);
__isl_give isl_map *isl_map_order_ge(__isl_take isl_map *map,
	enum isl_dim_type type1, int pos1, enum isl_dim_type type2, int pos2);
__isl_give isl_map *isl_map_order_le(__isl_take isl_map *map,
	enum isl_dim_type type1, int pos1, enum isl_dim_type type2, int pos2);
__isl_give isl_map *isl_map_equate(__isl_take isl_map *map,
````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_map_remove_unknown_divs`, `isl_map_remove_divs`, `isl_map_eliminate`, `isl_map_remove_dims` (+8 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_map_remove_unknown_divs`, `isl_map_remove_divs`, `isl_map_eliminate`, `isl_map_remove_dims` (+8 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 528-551

````cpp
	enum isl_dim_type type1, int pos1, enum isl_dim_type type2, int pos2);
__isl_give isl_map *isl_map_oppose(__isl_take isl_map *map,
	enum isl_dim_type type1, int pos1, enum isl_dim_type type2, int pos2);
__isl_give isl_map *isl_map_order_lt(__isl_take isl_map *map,
	enum isl_dim_type type1, int pos1, enum isl_dim_type type2, int pos2);
__isl_give isl_basic_map *isl_basic_map_order_gt(__isl_take isl_basic_map *bmap,
	enum isl_dim_type type1, int pos1, enum isl_dim_type type2, int pos2);
__isl_give isl_map *isl_map_order_gt(__isl_take isl_map *map,
	enum isl_dim_type type1, int pos1, enum isl_dim_type type2, int pos2);

__isl_export
__isl_give isl_map *isl_set_translation(__isl_take isl_set *deltas);
__isl_export
__isl_give isl_map *isl_set_identity(__isl_take isl_set *set);

__isl_export
isl_bool isl_basic_set_is_wrapping(__isl_keep isl_basic_set *bset);
__isl_export
isl_bool isl_set_is_wrapping(__isl_keep isl_set *set);
__isl_give isl_basic_set *isl_basic_map_wrap(__isl_take isl_basic_map *bmap);
__isl_export
__isl_give isl_set *isl_map_wrap(__isl_take isl_map *map);
__isl_give isl_basic_map *isl_basic_set_unwrap(__isl_take isl_basic_set *bset);
__isl_export
````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_map_oppose`, `isl_map_order_lt`, `isl_basic_map_order_gt`, `isl_map_order_gt` (+7 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_map_oppose`, `isl_map_order_lt`, `isl_basic_map_order_gt`, `isl_map_order_gt` (+7 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 552-575

````cpp
__isl_give isl_map *isl_set_unwrap(__isl_take isl_set *set);
__isl_export
__isl_give isl_basic_map *isl_basic_map_flatten(__isl_take isl_basic_map *bmap);
__isl_export
__isl_give isl_map *isl_map_flatten(__isl_take isl_map *map);
__isl_export
__isl_give isl_basic_map *isl_basic_map_flatten_domain(
	__isl_take isl_basic_map *bmap);
__isl_export
__isl_give isl_basic_map *isl_basic_map_flatten_range(
	__isl_take isl_basic_map *bmap);
__isl_export
__isl_give isl_map *isl_map_flatten_domain(__isl_take isl_map *map);
__isl_export
__isl_give isl_map *isl_map_flatten_range(__isl_take isl_map *map);
__isl_export
__isl_give isl_basic_set *isl_basic_set_flatten(__isl_take isl_basic_set *bset);
__isl_export
__isl_give isl_set *isl_set_flatten(__isl_take isl_set *set);
__isl_give isl_map *isl_set_flatten_map(__isl_take isl_set *set);
__isl_export
__isl_give isl_set *isl_map_params(__isl_take isl_map *map);
__isl_export
__isl_give isl_set *isl_map_domain(__isl_take isl_map *bmap);
````
- **EN**: This block declares or defines routines around `isl_set_unwrap`, `isl_basic_map_flatten`, `isl_map_flatten`, `isl_basic_map_flatten_domain` (+8 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_set_unwrap`, `isl_basic_map_flatten`, `isl_map_flatten`, `isl_basic_map_flatten_domain` (+8 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 576-598

````cpp
__isl_export
__isl_give isl_set *isl_map_range(__isl_take isl_map *map);
__isl_export
__isl_give isl_map *isl_set_insert_domain(__isl_take isl_set *set,
	__isl_take isl_space *domain);
__isl_give isl_map *isl_map_domain_map(__isl_take isl_map *map);
__isl_give isl_map *isl_map_range_map(__isl_take isl_map *map);
__isl_give isl_map *isl_set_wrapped_domain_map(__isl_take isl_set *set);
__isl_constructor
__isl_give isl_map *isl_map_from_basic_map(__isl_take isl_basic_map *bmap);
__isl_give isl_map *isl_map_from_domain(__isl_take isl_set *set);
__isl_give isl_basic_map *isl_basic_map_from_domain(
	__isl_take isl_basic_set *bset);
__isl_give isl_basic_map *isl_basic_map_from_range(
	__isl_take isl_basic_set *bset);
__isl_give isl_map *isl_map_from_range(__isl_take isl_set *set);
__isl_give isl_basic_map *isl_basic_map_from_domain_and_range(
	__isl_take isl_basic_set *domain, __isl_take isl_basic_set *range);
__isl_give isl_map *isl_map_from_domain_and_range(__isl_take isl_set *domain,
	__isl_take isl_set *range);
__isl_export
__isl_give isl_basic_map *isl_map_sample(__isl_take isl_map *map);

````
- **EN**: This block declares or defines routines around `isl_map_range`, `isl_set_insert_domain`, `isl_map_domain_map`, `isl_map_range_map` (+9 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_map_range`, `isl_set_insert_domain`, `isl_map_domain_map`, `isl_map_range_map` (+9 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 599-622

````cpp
__isl_export
__isl_give isl_set *isl_map_bind_domain(__isl_take isl_map *map,
	__isl_take isl_multi_id *tuple);
__isl_export
__isl_give isl_set *isl_map_bind_range(__isl_take isl_map *map,
	__isl_take isl_multi_id *tuple);

isl_bool isl_map_plain_is_empty(__isl_keep isl_map *map);
isl_bool isl_map_plain_is_universe(__isl_keep isl_map *map);
__isl_export
isl_bool isl_map_is_empty(__isl_keep isl_map *map);
__isl_export
isl_bool isl_map_is_subset(__isl_keep isl_map *map1, __isl_keep isl_map *map2);
__isl_export
isl_bool isl_map_is_strict_subset(__isl_keep isl_map *map1,
	__isl_keep isl_map *map2);
__isl_export
isl_bool isl_map_is_equal(__isl_keep isl_map *map1, __isl_keep isl_map *map2);
__isl_export
isl_bool isl_map_is_disjoint(__isl_keep isl_map *map1,
	__isl_keep isl_map *map2);
isl_bool isl_basic_map_is_single_valued(__isl_keep isl_basic_map *bmap);
isl_bool isl_map_plain_is_single_valued(__isl_keep isl_map *map);
__isl_export
````
- **EN**: This block declares or defines routines around `isl_map_bind_domain`, `isl_map_bind_range`, `isl_map_plain_is_empty`, `isl_map_plain_is_universe` (+7 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_map_bind_domain`, `isl_map_bind_range`, `isl_map_plain_is_empty`, `isl_map_plain_is_universe` (+7 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 623-645

````cpp
isl_bool isl_map_is_single_valued(__isl_keep isl_map *map);
isl_bool isl_map_plain_is_injective(__isl_keep isl_map *map);
__isl_export
isl_bool isl_map_is_injective(__isl_keep isl_map *map);
__isl_export
isl_bool isl_map_is_bijective(__isl_keep isl_map *map);
isl_bool isl_map_is_identity(__isl_keep isl_map *map);
int isl_map_is_translation(__isl_keep isl_map *map);
isl_bool isl_map_has_equal_space(__isl_keep isl_map *map1,
	__isl_keep isl_map *map2);

isl_bool isl_basic_map_can_zip(__isl_keep isl_basic_map *bmap);
isl_bool isl_map_can_zip(__isl_keep isl_map *map);
__isl_give isl_basic_map *isl_basic_map_zip(__isl_take isl_basic_map *bmap);
__isl_export
__isl_give isl_map *isl_map_zip(__isl_take isl_map *map);

isl_bool isl_basic_map_can_curry(__isl_keep isl_basic_map *bmap);
isl_bool isl_map_can_curry(__isl_keep isl_map *map);
__isl_give isl_basic_map *isl_basic_map_curry(__isl_take isl_basic_map *bmap);
__isl_export
__isl_give isl_map *isl_map_curry(__isl_take isl_map *map);

````
- **EN**: This block declares or defines routines around `isl_map_is_single_valued`, `isl_map_plain_is_injective`, `isl_map_is_injective`, `isl_map_is_bijective` (+11 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_map_is_single_valued`, `isl_map_plain_is_injective`, `isl_map_is_injective`, `isl_map_is_bijective` (+11 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 646-673

````cpp
isl_bool isl_map_can_range_curry(__isl_keep isl_map *map);
__isl_give isl_map *isl_map_range_curry(__isl_take isl_map *map);

isl_bool isl_basic_map_can_uncurry(__isl_keep isl_basic_map *bmap);
isl_bool isl_map_can_uncurry(__isl_keep isl_map *map);
__isl_give isl_basic_map *isl_basic_map_uncurry(__isl_take isl_basic_map *bmap);
__isl_export
__isl_give isl_map *isl_map_uncurry(__isl_take isl_map *map);

__isl_give isl_map *isl_map_make_disjoint(__isl_take isl_map *map);
__isl_give isl_map *isl_basic_map_compute_divs(__isl_take isl_basic_map *bmap);
__isl_give isl_map *isl_map_compute_divs(__isl_take isl_map *map);
ISL_DEPRECATED
__isl_give isl_map *isl_map_align_divs(__isl_take isl_map *map);

__isl_give isl_basic_map *isl_basic_map_drop_constraints_involving_dims(
	__isl_take isl_basic_map *bmap,
	enum isl_dim_type type, unsigned first, unsigned n);
__isl_give isl_basic_map *isl_basic_map_drop_constraints_not_involving_dims(
	__isl_take isl_basic_map *bmap,
	enum isl_dim_type type, unsigned first, unsigned n);
__isl_give isl_map *isl_map_drop_constraints_involving_dims(
	__isl_take isl_map *map,
	enum isl_dim_type type, unsigned first, unsigned n);
__isl_give isl_map *isl_map_drop_constraints_not_involving_dims(
	__isl_take isl_map *map,
	enum isl_dim_type type, unsigned first, unsigned n);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_map_can_range_curry`, `isl_map_range_curry`, `isl_basic_map_can_uncurry`, `isl_map_can_uncurry` (+10 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_map_can_range_curry`, `isl_map_range_curry`, `isl_basic_map_can_uncurry`, `isl_map_can_uncurry` (+10 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 674-697

````cpp
isl_bool isl_basic_map_involves_dims(__isl_keep isl_basic_map *bmap,
	enum isl_dim_type type, unsigned first, unsigned n);
isl_bool isl_map_involves_dims(__isl_keep isl_map *map,
	enum isl_dim_type type, unsigned first, unsigned n);

void isl_map_print_internal(__isl_keep isl_map *map, FILE *out, int indent);

__isl_give isl_val *isl_map_plain_get_val_if_fixed(__isl_keep isl_map *map,
	enum isl_dim_type type, unsigned pos);

__isl_give isl_basic_map *isl_basic_map_gist_domain(
	__isl_take isl_basic_map *bmap, __isl_take isl_basic_set *context);
__isl_export
__isl_give isl_basic_map *isl_basic_map_gist(__isl_take isl_basic_map *bmap,
	__isl_take isl_basic_map *context);
__isl_export
__isl_give isl_map *isl_map_gist(__isl_take isl_map *map,
	__isl_take isl_map *context);
__isl_export
__isl_give isl_map *isl_map_gist_domain(__isl_take isl_map *map,
	__isl_take isl_set *context);
__isl_give isl_map *isl_map_gist_range(__isl_take isl_map *map,
	__isl_take isl_set *context);
__isl_export
````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_basic_map_involves_dims`, `isl_map_involves_dims`, `isl_map_print_internal`, `isl_map_plain_get_val_if_fixed` (+5 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_basic_map_involves_dims`, `isl_map_involves_dims`, `isl_map_print_internal`, `isl_map_plain_get_val_if_fixed` (+5 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 698-717

````cpp
__isl_give isl_map *isl_map_gist_params(__isl_take isl_map *map,
	__isl_take isl_set *context);
__isl_give isl_map *isl_map_gist_basic_map(__isl_take isl_map *map,
	__isl_take isl_basic_map *context);

__isl_give isl_stride_info *isl_map_get_range_stride_info(
	__isl_keep isl_map *map, int pos);
__isl_export
__isl_give isl_fixed_box *isl_map_get_range_lattice_tile(
	__isl_keep isl_map *map);
__isl_export
__isl_give isl_fixed_box *isl_map_get_range_simple_fixed_box_hull(
	__isl_keep isl_map *map);

__isl_export
__isl_give isl_map *isl_map_coalesce(__isl_take isl_map *map);

isl_bool isl_map_plain_is_equal(__isl_keep isl_map *map1,
	__isl_keep isl_map *map2);

````
- **EN**: This block declares or defines routines around `isl_map_gist_params`, `isl_map_gist_basic_map`, `isl_map_get_range_stride_info`, `isl_map_get_range_lattice_tile` (+3 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_map_gist_params`, `isl_map_gist_basic_map`, `isl_map_get_range_stride_info`, `isl_map_get_range_lattice_tile` (+3 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 718-745

````cpp
uint32_t isl_map_get_hash(__isl_keep isl_map *map);

__isl_export
isl_size isl_map_n_basic_map(__isl_keep isl_map *map);
__isl_export
isl_stat isl_map_foreach_basic_map(__isl_keep isl_map *map,
	isl_stat (*fn)(__isl_take isl_basic_map *bmap, void *user), void *user);
__isl_give isl_basic_map_list *isl_map_get_basic_map_list(
	__isl_keep isl_map *map);

__isl_overload
__isl_give isl_map *isl_map_fixed_power_val(__isl_take isl_map *map,
	__isl_take isl_val *exp);
__isl_give isl_map *isl_map_power(__isl_take isl_map *map, isl_bool *exact);
__isl_give isl_map *isl_map_reaching_path_lengths(__isl_take isl_map *map,
	isl_bool *exact);
__isl_give isl_map *isl_map_transitive_closure(__isl_take isl_map *map,
	isl_bool *exact);

__isl_give isl_map *isl_map_lex_le_map(__isl_take isl_map *map1,
	__isl_take isl_map *map2);
__isl_give isl_map *isl_map_lex_lt_map(__isl_take isl_map *map1,
	__isl_take isl_map *map2);
__isl_give isl_map *isl_map_lex_ge_map(__isl_take isl_map *map1,
	__isl_take isl_map *map2);
__isl_give isl_map *isl_map_lex_gt_map(__isl_take isl_map *map1,
	__isl_take isl_map *map2);

````
- **EN**: This block declares or defines routines around `isl_map_get_hash`, `isl_map_n_basic_map`, `isl_map_foreach_basic_map`, `isl_stat` (+9 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_map_get_hash`, `isl_map_n_basic_map`, `isl_map_foreach_basic_map`, `isl_stat` (+9 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 746-770

````cpp
__isl_overload
__isl_give isl_map *isl_map_eq_at_multi_pw_aff(__isl_take isl_map *map,
	__isl_take isl_multi_pw_aff *mpa);
__isl_overload
__isl_give isl_map *isl_map_lex_lt_at_multi_pw_aff(__isl_take isl_map *map,
	__isl_take isl_multi_pw_aff *mpa);
__isl_overload
__isl_give isl_map *isl_map_lex_le_at_multi_pw_aff(__isl_take isl_map *map,
	__isl_take isl_multi_pw_aff *mpa);
__isl_overload
__isl_give isl_map *isl_map_lex_gt_at_multi_pw_aff(__isl_take isl_map *map,
	__isl_take isl_multi_pw_aff *mpa);
__isl_overload
__isl_give isl_map *isl_map_lex_ge_at_multi_pw_aff(__isl_take isl_map *map,
	__isl_take isl_multi_pw_aff *mpa);

__isl_give isl_basic_map *isl_basic_map_align_params(
	__isl_take isl_basic_map *bmap, __isl_take isl_space *model);
__isl_give isl_map *isl_map_align_params(__isl_take isl_map *map,
	__isl_take isl_space *model);
__isl_give isl_basic_map *isl_basic_map_drop_unused_params(
	__isl_take isl_basic_map *bmap);
__isl_export
__isl_give isl_map *isl_map_drop_unused_params(__isl_take isl_map *map);

````
- **EN**: This block declares or defines routines around `isl_map_eq_at_multi_pw_aff`, `isl_map_lex_lt_at_multi_pw_aff`, `isl_map_lex_le_at_multi_pw_aff`, `isl_map_lex_gt_at_multi_pw_aff` (+5 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_map_eq_at_multi_pw_aff`, `isl_map_lex_lt_at_multi_pw_aff`, `isl_map_lex_le_at_multi_pw_aff`, `isl_map_lex_gt_at_multi_pw_aff` (+5 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 771-790

````cpp
__isl_give isl_mat *isl_basic_map_equalities_matrix(
		__isl_keep isl_basic_map *bmap, enum isl_dim_type c1,
		enum isl_dim_type c2, enum isl_dim_type c3,
		enum isl_dim_type c4, enum isl_dim_type c5);
__isl_give isl_mat *isl_basic_map_inequalities_matrix(
		__isl_keep isl_basic_map *bmap, enum isl_dim_type c1,
		enum isl_dim_type c2, enum isl_dim_type c3,
		enum isl_dim_type c4, enum isl_dim_type c5);
__isl_give isl_basic_map *isl_basic_map_from_constraint_matrices(
	__isl_take isl_space *space,
	__isl_take isl_mat *eq, __isl_take isl_mat *ineq, enum isl_dim_type c1,
	enum isl_dim_type c2, enum isl_dim_type c3,
	enum isl_dim_type c4, enum isl_dim_type c5);

__isl_give isl_basic_map *isl_basic_map_from_aff(__isl_take isl_aff *aff);
__isl_give isl_basic_map *isl_basic_map_from_multi_aff(
	__isl_take isl_multi_aff *maff);
__isl_give isl_basic_map *isl_basic_map_from_aff_list(
	__isl_take isl_space *domain_space, __isl_take isl_aff_list *list);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_basic_map_equalities_matrix`, `isl_basic_map_inequalities_matrix`, `isl_basic_map_from_constraint_matrices`, `isl_basic_map_from_aff` (+2 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_basic_map_equalities_matrix`, `isl_basic_map_inequalities_matrix`, `isl_basic_map_from_constraint_matrices`, `isl_basic_map_from_aff` (+2 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 791-807

````cpp
__isl_give isl_map *isl_map_from_aff(__isl_take isl_aff *aff);
__isl_export
__isl_give isl_map *isl_multi_aff_as_map(__isl_take isl_multi_aff *ma);
__isl_give isl_map *isl_map_from_multi_aff(__isl_take isl_multi_aff *maff);

__isl_give isl_pw_aff *isl_map_dim_min(__isl_take isl_map *map, int pos);
__isl_give isl_pw_aff *isl_map_dim_max(__isl_take isl_map *map, int pos);

ISL_DECLARE_LIST_FN(basic_map)
ISL_DECLARE_EXPORTED_LIST_FN(map)
ISL_DECLARE_EXPORTED_LIST_FN_READ(map)

#if defined(__cplusplus)
}
#endif

#endif
````
- **EN**: This block uses preprocessor directives to control compilation, macros, or include guards; declares or defines routines around `isl_map_from_aff`, `isl_multi_aff_as_map`, `isl_map_from_multi_aff`, `isl_map_dim_min` (+4 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 使用预处理指令控制编译、宏或头文件保护; 声明或定义与 `isl_map_from_aff`, `isl_multi_aff_as_map`, `isl_map_from_multi_aff`, `isl_map_dim_min` (+4 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

## Key Concepts / 关键概念

- **ISL relations and sets**
  - **CN**: ISL 关系与集合
- **Loop transformation**
  - **CN**: 循环变换
- **Public interface declarations**
  - **CN**: 公共接口声明

## Dependencies / 依赖关系

- **ISL headers**: `isl/ctx.h`, `isl/space_type.h`, `isl/vec.h`, `isl/mat.h`, `isl/printer.h`, `isl/local_space.h`, `isl/aff_type.h`, `isl/list.h` (+5 more) — ISL relation/set primitives that support polyhedral analysis and transformations.
  **ISL headers（CN）**：`isl/ctx.h`, `isl/space_type.h`, `isl/vec.h`, `isl/mat.h`, `isl/printer.h`, `isl/local_space.h`, `isl/aff_type.h`, `isl/list.h` (+5 more) —— 支撑多面体分析与变换的 ISL 关系/集合原语。
- **System/standard headers**: `stdio.h` — Standard-library or system declarations required by the implementation.
  **System/standard headers（CN）**：`stdio.h` —— 实现所需的标准库或系统声明。
