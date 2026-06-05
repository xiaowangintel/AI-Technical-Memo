# isl_map_private.h — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `polly/lib/External/isl/isl_map_private.h` | `polly/lib/External/isl/isl_map_private.h` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | Implements or embeds upstream ISL support code used by Polly. | 实现或内嵌 Polly 使用的上游 ISL 支撑代码。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-26

````cpp
/*
 * Copyright 2008-2009 Katholieke Universiteit Leuven
 *
 * Use of this software is governed by the MIT license
 *
 * Written by Sven Verdoolaege, K.U.Leuven, Departement
 * Computerwetenschappen, Celestijnenlaan 200A, B-3001 Leuven, Belgium
 */

#ifndef ISL_MAP_PRIVATE_H
#define ISL_MAP_PRIVATE_H

#define isl_basic_set	isl_basic_map
#define isl_maybe_isl_basic_set	isl_maybe_isl_basic_map
#define isl_set		isl_map
#define isl_basic_set_list	isl_basic_map_list
#define isl_set_list	isl_map_list
#include <isl/list.h>
#include <isl/set.h>
#include <isl/map.h>
#include <isl_reordering.h>
#include <isl/vec.h>
#include <isl/hash.h>
#include <isl_blk.h>
#include <isl_maybe_aff.h>

````
- **EN**: This block imports ISL, system/standard headers needed by the surrounding code; uses preprocessor directives to control compilation, macros, or include guards; defines macros like `ISL_MAP_PRIVATE_H`, `isl_basic_set`, `isl_maybe_isl_basic_set`, `isl_set` (+2 more); uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 引入周边逻辑所需的 ISL、system/standard 头文件; 使用预处理指令控制编译、宏或头文件保护; 定义宏，例如 `ISL_MAP_PRIVATE_H`, `isl_basic_set`, `isl_maybe_isl_basic_set`, `isl_set` (+2 more); 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 27-50

````cpp
/* A "basic map" is a relation between two sets of variables,
 * called the "in" and "out" variables.
 * A "basic set" is a basic map with a zero-dimensional
 * domain.
 *
 * It is implemented as a set with two extra fields:
 * n_in is the number of in variables
 * n_out is the number of out variables
 * n_in + n_out should be equal to set.dim
 */
struct isl_basic_map {
	int ref;
#define ISL_BASIC_MAP_FINAL		(1 << 0)
#define ISL_BASIC_MAP_EMPTY		(1 << 1)
#define ISL_BASIC_MAP_NO_IMPLICIT	(1 << 2)
#define ISL_BASIC_MAP_NO_REDUNDANT	(1 << 3)
#define ISL_BASIC_MAP_RATIONAL		(1 << 4)
#define ISL_BASIC_MAP_SORTED		(1 << 5)
#define ISL_BASIC_MAP_NORMALIZED_DIVS	(1 << 6)
#define ISL_BASIC_MAP_ALL_EQUALITIES	(1 << 7)
#define ISL_BASIC_MAP_REDUCED_COEFFICIENTS	(1 << 8)
#define ISL_BASIC_SET_FINAL		(1 << 0)
#define ISL_BASIC_SET_EMPTY		(1 << 1)
#define ISL_BASIC_SET_NO_IMPLICIT	(1 << 2)
````
- **EN**: This block uses preprocessor directives to control compilation, macros, or include guards; declares or references types such as `isl_basic_map`; defines macros like `ISL_BASIC_MAP_FINAL`, `ISL_BASIC_MAP_EMPTY`, `ISL_BASIC_MAP_NO_IMPLICIT`, `ISL_BASIC_MAP_NO_REDUNDANT` (+8 more); uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 使用预处理指令控制编译、宏或头文件保护; 声明或引用类型，例如 `isl_basic_map`; 定义宏，例如 `ISL_BASIC_MAP_FINAL`, `ISL_BASIC_MAP_EMPTY`, `ISL_BASIC_MAP_NO_IMPLICIT`, `ISL_BASIC_MAP_NO_REDUNDANT` (+8 more); 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 51-70

````cpp
#define ISL_BASIC_SET_NO_REDUNDANT	(1 << 3)
#define ISL_BASIC_SET_RATIONAL		(1 << 4)
#define ISL_BASIC_SET_SORTED		(1 << 5)
#define ISL_BASIC_SET_NORMALIZED_DIVS	(1 << 6)
#define ISL_BASIC_SET_ALL_EQUALITIES	(1 << 7)
#define ISL_BASIC_SET_REDUCED_COEFFICIENTS	(1 << 8)
	unsigned flags;

	struct isl_ctx *ctx;

	isl_space *dim;
	unsigned extra;

	unsigned n_eq;
	unsigned n_ineq;

	size_t c_size;
	isl_int **eq;
	isl_int **ineq;

````
- **EN**: This block uses preprocessor directives to control compilation, macros, or include guards; declares or references types such as `isl_ctx`; defines macros like `ISL_BASIC_SET_NO_REDUNDANT`, `ISL_BASIC_SET_RATIONAL`, `ISL_BASIC_SET_SORTED`, `ISL_BASIC_SET_NORMALIZED_DIVS` (+2 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 使用预处理指令控制编译、宏或头文件保护; 声明或引用类型，例如 `isl_ctx`; 定义宏，例如 `ISL_BASIC_SET_NO_REDUNDANT`, `ISL_BASIC_SET_RATIONAL`, `ISL_BASIC_SET_SORTED`, `ISL_BASIC_SET_NORMALIZED_DIVS` (+2 more); 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 71-94

````cpp
	unsigned n_div;

	isl_int **div;

	struct isl_vec *sample;

	struct isl_blk block;
	struct isl_blk block2;
};

#undef EL
#define EL isl_basic_set

#include <isl_list_templ.h>

/* A "map" is a (possibly disjoint) union of basic maps.
 * A "set" is a (possibly disjoint) union of basic sets.
 *
 * Currently, the isl_set structure is identical to the isl_map structure
 * and the library depends on this correspondence internally.
 * However, users should not depend on this correspondence.
 *
 * "cached_simple_hull" contains copies of the unshifted and shifted
 * simple hulls, if they have already been computed.  Otherwise,
````
- **EN**: This block imports system/standard headers needed by the surrounding code; uses preprocessor directives to control compilation, macros, or include guards; declares or references types such as `isl_vec`, `isl_blk`; defines macros like `EL`; and continues the surrounding implementation details.
- **CN**: 该代码块 引入周边逻辑所需的 system/standard 头文件; 使用预处理指令控制编译、宏或头文件保护; 声明或引用类型，例如 `isl_vec`, `isl_blk`; 定义宏，例如 `EL`；并延续周边实现细节。

### Lines 95-115

````cpp
 * the entries are NULL.
 */
struct isl_map {
	int ref;
#define ISL_MAP_DISJOINT		(1 << 0)
#define ISL_MAP_NORMALIZED		(1 << 1)
#define ISL_SET_DISJOINT		(1 << 0)
#define ISL_SET_NORMALIZED		(1 << 1)
	unsigned flags;
	isl_basic_map *cached_simple_hull[2];

	struct isl_ctx *ctx;

	isl_space *dim;

	int n;

	size_t size;
	struct isl_basic_map *p[1];
};

````
- **EN**: This block uses preprocessor directives to control compilation, macros, or include guards; declares or references types such as `isl_map`, `isl_ctx`, `isl_basic_map`; defines macros like `ISL_MAP_DISJOINT`, `ISL_MAP_NORMALIZED`, `ISL_SET_DISJOINT`, `ISL_SET_NORMALIZED`; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 使用预处理指令控制编译、宏或头文件保护; 声明或引用类型，例如 `isl_map`, `isl_ctx`, `isl_basic_map`; 定义宏，例如 `ISL_MAP_DISJOINT`, `ISL_MAP_NORMALIZED`, `ISL_SET_DISJOINT`, `ISL_SET_NORMALIZED`; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 116-143

````cpp
#undef EL
#define EL isl_set

#include <isl_list_templ.h>

__isl_give isl_basic_set *isl_basic_set_alloc(isl_ctx *ctx,
	unsigned nparam, unsigned dim, unsigned extra,
	unsigned n_eq, unsigned n_ineq);
__isl_give isl_basic_set *isl_basic_set_extend_constraints(
	__isl_take isl_basic_set *base, unsigned n_eq, unsigned n_ineq);
__isl_give isl_basic_set *isl_basic_set_finalize(
	__isl_take isl_basic_set *bset);
__isl_give isl_basic_set *isl_basic_set_dup(__isl_keep isl_basic_set *bset);
__isl_give isl_basic_set *isl_basic_set_simplify(
	__isl_take isl_basic_set *bset);

__isl_give isl_basic_map *isl_basic_map_alloc(isl_ctx *ctx,
	unsigned nparam, unsigned in, unsigned out, unsigned extra,
	unsigned n_eq, unsigned n_ineq);
__isl_give isl_basic_map *isl_basic_map_mark_final(
	__isl_take isl_basic_map *bmap);
__isl_give isl_basic_map *isl_basic_map_finalize(
	__isl_take isl_basic_map *bmap);
__isl_give isl_basic_map *isl_basic_map_extend_constraints(
	__isl_take isl_basic_map *base, unsigned n_eq, unsigned n_ineq);
__isl_give isl_basic_map *isl_basic_map_simplify(
	__isl_take isl_basic_map *bmap);

````
- **EN**: This block imports system/standard headers needed by the surrounding code; uses preprocessor directives to control compilation, macros, or include guards; defines macros like `EL`; declares or defines routines around `isl_basic_set_alloc`, `isl_basic_set_extend_constraints`, `isl_basic_set_finalize`, `isl_basic_set_dup` (+6 more); and continues the surrounding implementation details.
- **CN**: 该代码块 引入周边逻辑所需的 system/standard 头文件; 使用预处理指令控制编译、宏或头文件保护; 定义宏，例如 `EL`; 声明或定义与 `isl_basic_set_alloc`, `isl_basic_set_extend_constraints`, `isl_basic_set_finalize`, `isl_basic_set_dup` (+6 more) 相关的例程；并延续周边实现细节。

### Lines 144-163

````cpp
__isl_give isl_set *isl_set_add_basic_set(__isl_take isl_set *set,
	__isl_take isl_basic_set *bset);

__isl_give isl_map *isl_map_add_basic_map(__isl_take isl_map *map,
	__isl_take isl_basic_map *bmap);
__isl_give isl_map *isl_map_dup(__isl_keep isl_map *map);

__isl_give isl_basic_set *isl_basic_set_from_underlying_set(
	__isl_take isl_basic_set *bset, __isl_take isl_basic_set *like);

__isl_give isl_map *isl_map_realign(__isl_take isl_map *map,
	__isl_take isl_reordering *r);
__isl_give isl_set *isl_set_realign(__isl_take isl_set *set,
	__isl_take isl_reordering *r);

__isl_give isl_basic_map *isl_basic_map_reset(__isl_take isl_basic_map *bmap,
	enum isl_dim_type type);
__isl_give isl_map *isl_map_reset(__isl_take isl_map *map,
	enum isl_dim_type type);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_set_add_basic_set`, `isl_map_add_basic_map`, `isl_map_dup`, `isl_basic_set_from_underlying_set` (+4 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_set_add_basic_set`, `isl_map_add_basic_map`, `isl_map_dup`, `isl_basic_set_from_underlying_set` (+4 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 164-187

````cpp
__isl_keep isl_space *isl_basic_map_peek_space(
	__isl_keep const isl_basic_map *bmap);
__isl_keep isl_space *isl_basic_set_peek_space(__isl_keep isl_basic_set *bset);
__isl_keep isl_space *isl_map_peek_space(__isl_keep const isl_map *map);
__isl_keep isl_space *isl_set_peek_space(__isl_keep isl_set *set);

__isl_give isl_basic_set *isl_basic_set_reset_space(
	__isl_take isl_basic_set *bset, __isl_take isl_space *space);
__isl_give isl_basic_map *isl_basic_map_reset_space(
	__isl_take isl_basic_map *bmap, __isl_take isl_space *space);
__isl_give isl_map *isl_map_reset_space(__isl_take isl_map *map,
	__isl_take isl_space *space);
__isl_give isl_map *isl_map_reset_equal_dim_space(__isl_take isl_map *map,
	__isl_take isl_space *space);

isl_size isl_basic_map_var_offset(__isl_keep isl_basic_map *bmap,
	enum isl_dim_type type);
isl_size isl_basic_set_var_offset(__isl_keep isl_basic_set *bset,
	enum isl_dim_type type);
unsigned isl_basic_map_offset(__isl_keep isl_basic_map *bmap,
					enum isl_dim_type type);
unsigned isl_basic_set_offset(__isl_keep isl_basic_set *bset,
					enum isl_dim_type type);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_basic_map_peek_space`, `isl_basic_set_peek_space`, `isl_map_peek_space`, `isl_set_peek_space` (+8 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_basic_map_peek_space`, `isl_basic_set_peek_space`, `isl_map_peek_space`, `isl_set_peek_space` (+8 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 188-209

````cpp
isl_bool isl_basic_map_may_be_set(__isl_keep isl_basic_map *bmap);
int isl_map_may_be_set(__isl_keep isl_map *map);
isl_bool isl_map_compatible_domain(__isl_keep isl_map *map,
	__isl_keep isl_set *set);
isl_bool isl_basic_map_compatible_domain(__isl_keep isl_basic_map *bmap,
	__isl_keep isl_basic_set *bset);
isl_bool isl_basic_map_compatible_range(__isl_keep isl_basic_map *bmap,
	__isl_keep isl_basic_set *bset);

__isl_give isl_basic_map *isl_basic_map_extend(__isl_take isl_basic_map *base,
	unsigned extra, unsigned n_eq, unsigned n_ineq);
__isl_give isl_basic_set *isl_basic_set_extend(__isl_take isl_basic_set *base,
	unsigned extra, unsigned n_eq, unsigned n_ineq);

__isl_give isl_map *isl_map_grow(__isl_take isl_map *map, int n);
__isl_give isl_set *isl_set_grow(__isl_take isl_set *set, int n);

isl_bool isl_basic_set_contains(__isl_keep isl_basic_set *bset,
	__isl_keep isl_vec *vec);
isl_bool isl_basic_map_contains(__isl_keep isl_basic_map *bmap,
	__isl_keep isl_vec *vec);

````
- **EN**: This block declares or defines routines around `isl_basic_map_may_be_set`, `isl_map_may_be_set`, `isl_map_compatible_domain`, `isl_basic_map_compatible_domain` (+7 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_basic_map_may_be_set`, `isl_map_may_be_set`, `isl_map_compatible_domain`, `isl_basic_map_compatible_domain` (+7 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 210-233

````cpp
__isl_give isl_basic_set *isl_basic_set_alloc_space(__isl_take isl_space *space,
		unsigned extra, unsigned n_eq, unsigned n_ineq);
__isl_give isl_set *isl_set_alloc_space(__isl_take isl_space *space, int n,
	unsigned flags);
__isl_give isl_basic_map *isl_basic_map_alloc_space(__isl_take isl_space *space,
		unsigned extra, unsigned n_eq, unsigned n_ineq);
__isl_give isl_map *isl_map_alloc_space(__isl_take isl_space *space, int n,
	unsigned flags);

int isl_basic_map_alloc_equality(__isl_keep isl_basic_map *bmap);
int isl_basic_set_alloc_equality(__isl_keep isl_basic_set *bset);
__isl_give isl_basic_set *isl_basic_set_free_inequality(
	__isl_take isl_basic_set *bset, unsigned n);
__isl_give isl_basic_map *isl_basic_map_free_equality(
	__isl_take isl_basic_map *bmap, unsigned n);
__isl_give isl_basic_set *isl_basic_set_free_equality(
	__isl_take isl_basic_set *bset, unsigned n);
int isl_basic_set_alloc_inequality(__isl_keep isl_basic_set *bset);
int isl_basic_map_alloc_inequality(__isl_keep isl_basic_map *bmap);
__isl_give isl_basic_map *isl_basic_map_free_inequality(
	__isl_take isl_basic_map *bmap, unsigned n);
int isl_basic_map_alloc_div(__isl_keep isl_basic_map *bmap);
__isl_give isl_basic_map *isl_basic_map_insert_div(
	__isl_take isl_basic_map *bmap, int pos, __isl_keep isl_vec *div);
````
- **EN**: This block declares or defines routines around `isl_basic_set_alloc_space`, `isl_set_alloc_space`, `isl_basic_map_alloc_space`, `isl_map_alloc_space` (+10 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_basic_set_alloc_space`, `isl_set_alloc_space`, `isl_basic_map_alloc_space`, `isl_map_alloc_space` (+10 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 234-253

````cpp
__isl_give isl_basic_set *isl_basic_set_insert_div(
	__isl_take isl_basic_set *bset, int pos, __isl_keep isl_vec *div);
int isl_basic_set_alloc_div(__isl_keep isl_basic_set *bset);
isl_stat isl_basic_map_free_div(__isl_keep isl_basic_map *bmap, unsigned n);
__isl_give isl_basic_map *isl_basic_map_drop_div(
	__isl_take isl_basic_map *bmap, unsigned div);
void isl_basic_map_inequality_to_equality(
		__isl_keep isl_basic_map *bmap, unsigned pos);
int isl_basic_map_drop_equality(__isl_keep isl_basic_map *bmap, unsigned pos);
int isl_basic_set_drop_inequality(__isl_keep isl_basic_set *bset, unsigned pos);
int isl_basic_map_drop_inequality(__isl_keep isl_basic_map *bmap, unsigned pos);
__isl_give isl_basic_set *isl_basic_set_add_eq(__isl_take isl_basic_set *bset,
	isl_int *eq);
__isl_give isl_basic_map *isl_basic_map_add_eq(__isl_take isl_basic_map *bmap,
	isl_int *eq);
__isl_give isl_basic_set *isl_basic_set_add_ineq(__isl_take isl_basic_set *bset,
	isl_int *ineq);
__isl_give isl_basic_map *isl_basic_map_add_ineq(__isl_take isl_basic_map *bmap,
	isl_int *ineq);

````
- **EN**: This block declares or defines routines around `isl_basic_set_insert_div`, `isl_basic_set_alloc_div`, `isl_basic_map_free_div`, `isl_basic_map_drop_div` (+8 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_basic_set_insert_div`, `isl_basic_set_alloc_div`, `isl_basic_map_free_div`, `isl_basic_map_drop_div` (+8 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 254-277

````cpp
__isl_give isl_basic_set *isl_basic_set_tighten_outward(
	__isl_take isl_basic_set *bset, __isl_keep isl_vec *vec);

__isl_give isl_basic_map *isl_inequality_negate(__isl_take isl_basic_map *bmap,
	unsigned pos);

isl_bool isl_basic_map_has_single_reference(__isl_keep isl_basic_map *bmap);
__isl_give isl_basic_set *isl_basic_set_cow(__isl_take isl_basic_set *bset);
__isl_give isl_basic_map *isl_basic_map_cow(__isl_take isl_basic_map *bmap);
__isl_give isl_set *isl_set_cow(__isl_take isl_set *set);
__isl_give isl_map *isl_map_cow(__isl_take isl_map *map);

uint32_t isl_basic_map_get_hash(__isl_keep isl_basic_map *bmap);

__isl_give isl_set *isl_basic_set_list_union(
	__isl_take isl_basic_set_list *list);

__isl_give isl_basic_map *isl_basic_map_set_to_empty(
	__isl_take isl_basic_map *bmap);
__isl_give isl_basic_set *isl_basic_set_set_to_empty(
	__isl_take isl_basic_set *bset);
__isl_give isl_basic_map *isl_basic_map_swap_div(__isl_take isl_basic_map *bmap,
	int a, int b);
__isl_give isl_basic_map *isl_basic_map_order_divs(
````
- **EN**: This block declares or defines routines around `isl_basic_set_tighten_outward`, `isl_inequality_negate`, `isl_basic_map_has_single_reference`, `isl_basic_set_cow` (+9 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_basic_set_tighten_outward`, `isl_inequality_negate`, `isl_basic_map_has_single_reference`, `isl_basic_set_cow` (+9 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 278-301

````cpp
	__isl_take isl_basic_map *bmap);
__isl_give isl_map *isl_map_order_divs(__isl_take isl_map *map);
__isl_give isl_basic_map *isl_basic_map_align_divs(
	__isl_take isl_basic_map *dst, __isl_keep isl_basic_map *src);
__isl_give isl_map *isl_map_align_divs_to_basic_map_list(
	__isl_take isl_map *map, __isl_keep isl_basic_map_list *list);
__isl_give isl_basic_map_list *isl_basic_map_list_align_divs_to_basic_map(
	__isl_take isl_basic_map_list *list, __isl_keep isl_basic_map *bmap);
__isl_give isl_map *isl_map_align_divs_internal(__isl_take isl_map *map);
__isl_give isl_basic_set *isl_basic_set_sort_divs(
	__isl_take isl_basic_set *bset);
__isl_give isl_basic_map *isl_basic_map_sort_divs(
	__isl_take isl_basic_map *bmap);
__isl_give isl_map *isl_map_sort_divs(__isl_take isl_map *map);
__isl_give isl_basic_map *isl_basic_map_gauss5(__isl_take isl_basic_map *bmap,
	int *progress,
	isl_stat (*swap)(unsigned a, unsigned b, void *user),
	isl_stat (*drop)(unsigned n, void *user), void *user);
__isl_give isl_basic_map *isl_basic_map_gauss(__isl_take isl_basic_map *bmap,
	int *progress);
__isl_give isl_basic_set *isl_basic_set_gauss(
	__isl_take isl_basic_set *bset, int *progress);
int isl_basic_map_constraint_cmp(__isl_keep isl_basic_map *bmap,
	isl_int *c1, isl_int *c2);
````
- **EN**: This block declares or defines routines around `isl_map_order_divs`, `isl_basic_map_align_divs`, `isl_map_align_divs_to_basic_map_list`, `isl_basic_map_list_align_divs_to_basic_map` (+9 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_map_order_divs`, `isl_basic_map_align_divs`, `isl_map_align_divs_to_basic_map_list`, `isl_basic_map_list_align_divs_to_basic_map` (+9 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 302-325

````cpp
__isl_give isl_basic_map *isl_basic_map_sort_constraints(
	__isl_take isl_basic_map *bmap);
__isl_give isl_basic_set *isl_basic_set_sort_constraints(
	__isl_take isl_basic_set *bset);
int isl_basic_map_plain_cmp(__isl_keep isl_basic_map *bmap1,
	__isl_keep isl_basic_map *bmap2);
isl_bool isl_basic_map_plain_is_equal(__isl_keep isl_basic_map *bmap1,
	__isl_keep isl_basic_map *bmap2);
__isl_give isl_basic_map *isl_basic_map_normalize_constraints(
	__isl_take isl_basic_map *bmap);
__isl_give isl_basic_set *isl_basic_set_normalize_constraints(
	__isl_take isl_basic_set *bset);
__isl_give isl_basic_map *isl_basic_map_implicit_equalities(
	__isl_take isl_basic_map *bmap);
__isl_give isl_basic_set *isl_basic_map_underlying_set(
	__isl_take isl_basic_map *bmap);
__isl_give isl_basic_set *isl_basic_set_underlying_set(
		__isl_take isl_basic_set *bset);
__isl_give isl_basic_set_list *isl_basic_map_list_underlying_set(
	__isl_take isl_basic_map_list *list);
__isl_give isl_set *isl_map_underlying_set(__isl_take isl_map *map);
__isl_give isl_basic_map *isl_basic_map_overlying_set(
	__isl_take isl_basic_set *bset, __isl_take isl_basic_map *like);
__isl_give isl_basic_set *isl_basic_set_drop_constraints_involving_unknown_divs(
````
- **EN**: This block declares or defines routines around `isl_basic_map_sort_constraints`, `isl_basic_set_sort_constraints`, `isl_basic_map_plain_cmp`, `isl_basic_map_plain_is_equal` (+9 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_basic_map_sort_constraints`, `isl_basic_set_sort_constraints`, `isl_basic_map_plain_cmp`, `isl_basic_map_plain_is_equal` (+9 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 326-347

````cpp
	__isl_take isl_basic_set *bset);
__isl_give isl_basic_map *isl_basic_map_drop_constraints_involving_unknown_divs(
	__isl_take isl_basic_map *bmap);
__isl_give isl_map *isl_map_drop_constraints_involving_unknown_divs(
	__isl_take isl_map *map);
__isl_give isl_basic_map *isl_basic_map_drop_constraints_involving(
	__isl_take isl_basic_map *bmap, unsigned first, unsigned n);
__isl_give isl_basic_set *isl_basic_set_drop_constraints_involving(
	__isl_take isl_basic_set *bset, unsigned first, unsigned n);
__isl_give isl_basic_set *isl_basic_set_drop(__isl_take isl_basic_set *bset,
	enum isl_dim_type type, unsigned first, unsigned n);
__isl_give isl_basic_map *isl_basic_map_drop(__isl_take isl_basic_map *bmap,
	enum isl_dim_type type, unsigned first, unsigned n);
__isl_give isl_set *isl_set_drop(__isl_take isl_set *set,
	enum isl_dim_type type, unsigned first, unsigned n);
__isl_give isl_basic_set *isl_basic_set_drop_dims(
	__isl_take isl_basic_set *bset, unsigned first, unsigned n);
__isl_give isl_map *isl_map_drop(__isl_take isl_map *map,
	enum isl_dim_type type, unsigned first, unsigned n);
__isl_give isl_basic_map *isl_basic_map_drop_unrelated_constraints(
	__isl_take isl_basic_map *bmap, __isl_take int *group);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_basic_map_drop_constraints_involving_unknown_divs`, `isl_map_drop_constraints_involving_unknown_divs`, `isl_basic_map_drop_constraints_involving`, `isl_basic_set_drop_constraints_involving` (+6 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_basic_map_drop_constraints_involving_unknown_divs`, `isl_map_drop_constraints_involving_unknown_divs`, `isl_basic_map_drop_constraints_involving`, `isl_basic_set_drop_constraints_involving` (+6 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 348-368

````cpp
__isl_give isl_basic_map *isl_basic_map_remove_unknown_divs(
	__isl_take isl_basic_map *bmap);

__isl_give isl_basic_map *isl_basic_map_eliminate_pure_unit_divs(
	__isl_take isl_basic_map *bmap);
__isl_give isl_basic_map *isl_basic_map_remove_duplicate_constraints(
	__isl_take isl_basic_map *bmap, int *progress, int detect_divs);
__isl_give isl_basic_map *isl_basic_map_detect_inequality_pairs(
	__isl_take isl_basic_map *bmap, int *progress);

__isl_give isl_map *isl_map_remove_empty_parts(__isl_take isl_map *map);
__isl_give isl_set *isl_set_remove_empty_parts(__isl_take isl_set *set);
__isl_give isl_map *isl_map_remove_obvious_duplicates(__isl_take isl_map *map);

__isl_give isl_set *isl_set_normalize(__isl_take isl_set *set);

__isl_give isl_basic_map *isl_basic_map_eliminate_vars(
	__isl_take isl_basic_map *bmap, unsigned pos, unsigned n);
__isl_give isl_basic_set *isl_basic_set_eliminate_vars(
	__isl_take isl_basic_set *bset, unsigned pos, unsigned n);

````
- **EN**: This block declares or defines routines around `isl_basic_map_remove_unknown_divs`, `isl_basic_map_eliminate_pure_unit_divs`, `isl_basic_map_remove_duplicate_constraints`, `isl_basic_map_detect_inequality_pairs` (+6 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_basic_map_remove_unknown_divs`, `isl_basic_map_eliminate_pure_unit_divs`, `isl_basic_map_remove_duplicate_constraints`, `isl_basic_map_detect_inequality_pairs` (+6 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 369-393

````cpp
__isl_give isl_map *isl_map_eliminate(__isl_take isl_map *map,
	enum isl_dim_type type, unsigned first, unsigned n);
__isl_give isl_set *isl_set_eliminate(__isl_take isl_set *set,
	enum isl_dim_type type, unsigned first, unsigned n);
__isl_give isl_map *isl_map_project_onto(__isl_take isl_map *map,
	enum isl_dim_type type, unsigned first, unsigned n);

__isl_give isl_basic_map *isl_basic_map_add_div_constraint(
	__isl_take isl_basic_map *bmap, unsigned div, int sign);
__isl_give isl_basic_map *isl_basic_map_add_div_constraints(
	__isl_take isl_basic_map *bmap, unsigned div);
__isl_give isl_basic_set *isl_basic_set_add_div_constraints(
	__isl_take isl_basic_set *bset, unsigned pos);
__isl_give isl_basic_map *isl_basic_map_add_known_div_constraints(
	__isl_take isl_basic_map *bmap);
__isl_give isl_basic_map *isl_basic_map_drop_redundant_divs(
	__isl_take isl_basic_map *bmap);

__isl_give isl_basic_set *isl_basic_set_recession_cone(
	__isl_take isl_basic_set *bset);
__isl_give isl_basic_set *isl_basic_set_lineality_space(
	__isl_take isl_basic_set *bset);
__isl_give isl_basic_set *isl_set_combined_lineality_space(
	__isl_take isl_set *set);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_map_eliminate`, `isl_set_eliminate`, `isl_map_project_onto`, `isl_basic_map_add_div_constraint` (+7 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_map_eliminate`, `isl_set_eliminate`, `isl_map_project_onto`, `isl_basic_map_add_div_constraint` (+7 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 394-416

````cpp
__isl_give isl_basic_set *isl_basic_set_set_integral(
	__isl_take isl_basic_set *bset);
__isl_give isl_basic_set *isl_basic_set_set_rational(
	__isl_take isl_basic_set *bset);
__isl_give isl_set *isl_set_set_rational(__isl_take isl_set *set);
__isl_give isl_basic_map *isl_basic_map_set_rational(
	__isl_take isl_basic_map *bmap);
__isl_give isl_map *isl_map_set_rational(__isl_take isl_map *map);

isl_bool isl_map_is_rational(__isl_keep isl_map *map);
isl_bool isl_set_is_rational(__isl_keep isl_set *set);

isl_bool isl_map_has_rational(__isl_keep isl_map *map);
isl_bool isl_set_has_rational(__isl_keep isl_set *set);

__isl_give isl_basic_map *isl_basic_map_from_multi_aff2(
	__isl_take isl_multi_aff *maff, int rational);
__isl_give isl_map *isl_map_from_multi_aff_internal(
	__isl_take isl_multi_aff *ma);
__isl_give isl_map *isl_map_from_pw_aff_internal(__isl_take isl_pw_aff *pa);
__isl_give isl_map *isl_map_from_pw_multi_aff_internal(
	__isl_take isl_pw_multi_aff *pma);

````
- **EN**: This block declares or defines routines around `isl_basic_set_set_integral`, `isl_basic_set_set_rational`, `isl_set_set_rational`, `isl_basic_map_set_rational` (+9 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_basic_set_set_integral`, `isl_basic_set_set_rational`, `isl_set_set_rational`, `isl_basic_map_set_rational` (+9 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 417-438

````cpp
struct isl_mat;

__isl_give isl_basic_set *isl_basic_set_preimage(
	__isl_take isl_basic_set *bset, __isl_take isl_mat *mat);
__isl_give isl_set *isl_set_preimage(
	__isl_take isl_set *set, __isl_take isl_mat *mat);

__isl_give isl_basic_map *isl_basic_map_transform_dims(
	__isl_take isl_basic_map *bmap, enum isl_dim_type type, unsigned first,
	__isl_take isl_mat *trans);
__isl_give isl_basic_set *isl_basic_set_transform_dims(
	__isl_take isl_basic_set *bset, enum isl_dim_type type, unsigned first,
	__isl_take isl_mat *trans);

isl_int *isl_set_wrap_facet(__isl_keep isl_set *set,
	isl_int *facet, isl_int *ridge);

isl_bool isl_basic_map_contains_point(__isl_keep isl_basic_map *bmap,
	__isl_keep isl_point *point);
isl_bool isl_set_contains_point(__isl_keep isl_set *set,
	__isl_keep isl_point *point);

````
- **EN**: This block declares or references types such as `isl_mat`; declares or defines routines around `isl_basic_set_preimage`, `isl_set_preimage`, `isl_basic_map_transform_dims`, `isl_basic_set_transform_dims` (+3 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或引用类型，例如 `isl_mat`; 声明或定义与 `isl_basic_set_preimage`, `isl_set_preimage`, `isl_basic_map_transform_dims`, `isl_basic_set_transform_dims` (+3 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 439-458

````cpp
isl_stat isl_basic_set_vars_get_sign(__isl_keep isl_basic_set *bset,
	unsigned first, unsigned n, int *signs);
isl_stat isl_set_foreach_orthant(__isl_keep isl_set *set,
	isl_stat (*fn)(__isl_take isl_set *orthant, int *signs, void *user),
	void *user);

isl_bool isl_basic_set_eq_is_stride(__isl_keep isl_basic_set *bset, int i);

isl_bool isl_basic_map_is_div_constraint(__isl_keep isl_basic_map *bmap,
	isl_int *constraint, unsigned div);

__isl_give isl_basic_set *isl_basic_set_from_local_space(
	__isl_take isl_local_space *ls);
__isl_give isl_basic_map *isl_basic_map_from_local_space(
	__isl_take isl_local_space *ls);
__isl_give isl_basic_set *isl_basic_set_expand_divs(
	__isl_take isl_basic_set *bset, __isl_take isl_mat *div, int *exp);
__isl_give isl_basic_map *isl_basic_map_expand_divs(
	__isl_take isl_basic_set *bmap, __isl_take isl_mat *div, int *exp);

````
- **EN**: This block declares or defines routines around `isl_basic_set_vars_get_sign`, `isl_set_foreach_orthant`, `isl_stat`, `isl_basic_set_eq_is_stride` (+5 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_basic_set_vars_get_sign`, `isl_set_foreach_orthant`, `isl_stat`, `isl_basic_set_eq_is_stride` (+5 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 459-483

````cpp
isl_size isl_basic_set_n_equality(__isl_keep isl_basic_set *bset);
isl_size isl_basic_map_n_equality(__isl_keep isl_basic_map *bmap);
isl_size isl_basic_set_n_inequality(__isl_keep isl_basic_set *bset);
isl_size isl_basic_map_n_inequality(__isl_keep isl_basic_map *bmap);

isl_bool isl_basic_map_div_is_integral(__isl_keep isl_basic_map *bmap, int div);

__isl_give isl_basic_map *isl_basic_map_mark_div_unknown(
	__isl_take isl_basic_map *bmap, int div);
isl_bool isl_basic_map_div_is_marked_unknown(__isl_keep isl_basic_map *bmap,
	int div);
isl_bool isl_basic_map_div_is_known(__isl_keep isl_basic_map *bmap, int div);
int isl_basic_set_first_unknown_div(__isl_keep isl_basic_set *bset);
int isl_basic_map_first_unknown_div(__isl_keep isl_basic_map *bmap);
isl_bool isl_basic_map_divs_known(__isl_keep isl_basic_map *bmap);
isl_bool isl_map_divs_known(__isl_keep isl_map *map);
__isl_give isl_mat *isl_basic_set_get_divs(__isl_keep isl_basic_set *bset);
__isl_give isl_mat *isl_basic_map_get_divs(__isl_keep isl_basic_map *bmap);

isl_bool isl_set_every_basic_set(__isl_keep isl_set *set,
	isl_bool (*test)(__isl_keep isl_basic_set *bset, void *user),
	void *user);
__isl_give isl_map *isl_map_inline_foreach_basic_map(__isl_take isl_map *map,
	__isl_give isl_basic_map *(*fn)(__isl_take isl_basic_map *bmap));

````
- **EN**: This block declares or defines routines around `isl_basic_set_n_equality`, `isl_basic_map_n_equality`, `isl_basic_set_n_inequality`, `isl_basic_map_n_inequality` (+13 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_basic_set_n_equality`, `isl_basic_map_n_equality`, `isl_basic_set_n_inequality`, `isl_basic_map_n_inequality` (+13 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 484-503

````cpp
isl_stat isl_basic_set_check_no_params(__isl_keep isl_basic_set *bset);
isl_stat isl_basic_set_check_no_locals(__isl_keep isl_basic_set *bset);

isl_stat isl_basic_set_check_range(__isl_keep isl_basic_set *bset,
	enum isl_dim_type type, unsigned first, unsigned n);
isl_stat isl_set_check_range(__isl_keep isl_set *set,
	enum isl_dim_type type, unsigned first, unsigned n);
isl_stat isl_basic_map_check_range(__isl_keep isl_basic_map *bmap,
	enum isl_dim_type type, unsigned first, unsigned n);
isl_stat isl_map_check_named_params(__isl_keep isl_map *map);

isl_bool isl_map_has_equal_params(__isl_keep isl_map *map1,
	__isl_keep isl_map *map2);
isl_bool isl_basic_set_space_has_equal_params(__isl_keep isl_basic_set *bset,
	__isl_keep isl_space *space);
isl_bool isl_set_space_has_equal_params(__isl_keep isl_set *set,
	__isl_keep isl_space *space);
isl_bool isl_map_space_has_equal_params(__isl_keep isl_map *map,
	__isl_keep isl_space *space);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_basic_set_check_no_params`, `isl_basic_set_check_no_locals`, `isl_basic_set_check_range`, `isl_set_check_range` (+6 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_basic_set_check_no_params`, `isl_basic_set_check_no_locals`, `isl_basic_set_check_range`, `isl_set_check_range` (+6 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 504-523

````cpp
isl_stat isl_map_align_params_bin(__isl_keep isl_map **map1,
	__isl_keep isl_map **map2);
isl_stat isl_map_align_params_set(__isl_keep isl_map **map,
	__isl_keep isl_set **set);
isl_bool isl_map_align_params_map_map_and_test(__isl_keep isl_map *map1,
	__isl_keep isl_map *map2,
	isl_bool (*fn)(__isl_keep isl_map *map1, __isl_keep isl_map *map2));

__isl_give isl_set *isl_set_substitute(__isl_take isl_set *set,
	unsigned pos, __isl_keep isl_aff *subs);

__isl_give isl_set *isl_set_gist_params_basic_set(__isl_take isl_set *set,
	__isl_take isl_basic_set *context);

isl_bool isl_map_compatible_range(__isl_keep isl_map *map,
	__isl_keep isl_set *set);

isl_bool isl_basic_map_plain_is_non_empty(__isl_keep isl_basic_map *bmap);
isl_bool isl_basic_map_plain_is_single_valued(__isl_keep isl_basic_map *bmap);

````
- **EN**: This block declares or defines routines around `isl_map_align_params_bin`, `isl_map_align_params_set`, `isl_map_align_params_map_map_and_test`, `isl_bool` (+5 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_map_align_params_bin`, `isl_map_align_params_set`, `isl_map_align_params_map_map_and_test`, `isl_bool` (+5 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 524-547

````cpp
isl_bool isl_basic_map_is_set(__isl_keep isl_basic_map *bmap);
isl_bool isl_map_is_set(__isl_keep isl_map *map);
isl_bool isl_map_is_params(__isl_keep isl_map *map);

isl_bool isl_basic_set_plain_dim_is_fixed(__isl_keep isl_basic_set *bset,
	unsigned dim, isl_int *val);

__isl_give isl_set *isl_set_plain_gist_basic_set(__isl_take isl_set *set,
	__isl_take isl_basic_set *context);
__isl_give isl_map *isl_map_plain_gist_basic_map(__isl_take isl_map *map,
	__isl_take isl_basic_map *context);
__isl_give isl_map *isl_map_plain_gist(__isl_take isl_map *map,
	__isl_take isl_map *context);

__isl_give isl_basic_set *isl_basic_set_plain_affine_hull(
	__isl_take isl_basic_set *bset);
__isl_give isl_basic_map *isl_basic_map_plain_affine_hull(
	__isl_take isl_basic_map *bmap);

isl_stat isl_basic_set_dim_residue_class(__isl_keep isl_basic_set *bset,
	int pos, isl_int *modulo, isl_int *residue);
isl_stat isl_set_dim_residue_class(__isl_keep isl_set *set,
	int pos, isl_int *modulo, isl_int *residue);

````
- **EN**: This block declares or defines routines around `isl_basic_map_is_set`, `isl_map_is_set`, `isl_map_is_params`, `isl_basic_set_plain_dim_is_fixed` (+7 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_basic_map_is_set`, `isl_map_is_set`, `isl_map_is_params`, `isl_basic_set_plain_dim_is_fixed` (+7 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 548-569

````cpp
__isl_give isl_basic_set *isl_basic_set_fix(__isl_take isl_basic_set *bset,
	enum isl_dim_type type, unsigned pos, isl_int value);
__isl_give isl_basic_map *isl_basic_map_fix(__isl_take isl_basic_map *bmap,
	enum isl_dim_type type, unsigned pos, isl_int value);
__isl_give isl_set *isl_set_fix(__isl_take isl_set *set,
	enum isl_dim_type type, unsigned pos, isl_int value);
isl_bool isl_map_plain_is_fixed(__isl_keep isl_map *map,
	enum isl_dim_type type, unsigned pos, isl_int *val);

int isl_basic_map_output_defining_equality(__isl_keep isl_basic_map *bmap,
	int pos, int *div, int *ineq);

__isl_give isl_basic_map *isl_basic_map_reduce_coefficients(
	__isl_take isl_basic_map *bmap);

__isl_give isl_basic_map *isl_basic_map_shift_div(
	__isl_take isl_basic_map *bmap, int div, int pos, isl_int shift);

int isl_basic_set_count_upto(__isl_keep isl_basic_set *bset,
	isl_int max, isl_int *count);
int isl_set_count_upto(__isl_keep isl_set *set, isl_int max, isl_int *count);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_basic_set_fix`, `isl_basic_map_fix`, `isl_set_fix`, `isl_map_plain_is_fixed` (+5 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_basic_set_fix`, `isl_basic_map_fix`, `isl_set_fix`, `isl_map_plain_is_fixed` (+5 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 570-593

````cpp
isl_bool isl_map_space_tuple_is_equal(__isl_keep isl_map *map,
	enum isl_dim_type type1, __isl_keep isl_space *space,
	enum isl_dim_type type2);
isl_bool isl_map_tuple_is_equal(__isl_keep isl_map *map1,
	enum isl_dim_type type1, __isl_keep isl_map *map2,
	enum isl_dim_type type2);
isl_bool isl_map_has_space(__isl_keep isl_map *map,
	__isl_keep isl_space *space);
isl_bool isl_map_has_space_tuples(__isl_keep isl_map *map,
	__isl_keep isl_space *space);

isl_bool isl_basic_map_is_transformation(__isl_keep isl_basic_map *bmap);
isl_stat isl_map_check_transformation(__isl_keep isl_map *map);
isl_stat isl_basic_set_check_equal_space(__isl_keep isl_basic_set *bset1,
	__isl_keep isl_basic_set *bset2);
isl_stat isl_basic_map_check_equal_space(__isl_keep isl_basic_map *bmap1,
	__isl_keep isl_basic_map *bmap2);
isl_stat isl_set_basic_set_check_equal_space(__isl_keep isl_set *set,
	__isl_keep isl_basic_set *bset);
isl_stat isl_map_basic_map_check_equal_space(__isl_keep isl_map *map,
	__isl_keep isl_basic_map *bmap);
isl_stat isl_map_check_equal_space(__isl_keep isl_map *map1,
	__isl_keep isl_map *map2);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_map_space_tuple_is_equal`, `isl_map_tuple_is_equal`, `isl_map_has_space`, `isl_map_has_space_tuples` (+7 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_map_space_tuple_is_equal`, `isl_map_tuple_is_equal`, `isl_map_has_space`, `isl_map_has_space_tuples` (+7 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 594-613

````cpp
isl_bool isl_basic_map_applies_range(__isl_keep isl_basic_map *bmap1,
	__isl_keep isl_basic_map *bmap2);

__isl_give isl_vec *isl_basic_map_inequality_extract_output_upper_bound(
	__isl_keep isl_basic_map *bmap, int ineq, int pos);

isl_bool isl_basic_map_div_expr_involves_vars(__isl_keep isl_basic_map *bmap,
	int div, unsigned first, unsigned n);
isl_bool isl_basic_map_any_div_involves_vars(__isl_keep isl_basic_map *bmap,
	unsigned first, unsigned n);

isl_size isl_basic_map_find_output_upper_div_constraint(
	__isl_keep isl_basic_map *bmap, int pos);
__isl_give isl_maybe_isl_aff isl_basic_map_try_find_output_mod(
	__isl_keep isl_basic_map *bmap, int pos);
__isl_give isl_maybe_isl_aff isl_basic_map_try_find_output_div_mod(
	__isl_keep isl_basic_map *bmap, int pos);
__isl_give isl_maybe_isl_aff isl_basic_map_try_find_any_output_div_mod(
	__isl_keep isl_basic_map *bmap, int *pos);

````
- **EN**: This block declares or defines routines around `isl_basic_map_applies_range`, `isl_basic_map_inequality_extract_output_upper_bound`, `isl_basic_map_div_expr_involves_vars`, `isl_basic_map_any_div_involves_vars` (+4 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_basic_map_applies_range`, `isl_basic_map_inequality_extract_output_upper_bound`, `isl_basic_map_div_expr_involves_vars`, `isl_basic_map_any_div_involves_vars` (+4 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 614-626

````cpp
__isl_give isl_mat *isl_basic_set_extract_equalities(
	__isl_keep isl_basic_set *bset);

isl_bool isl_basic_map_equal_div_expr_part(__isl_keep isl_basic_map *bmap1,
	int pos1, __isl_keep isl_basic_map *bmap2, int pos2,
	unsigned first, unsigned n);
isl_bool isl_basic_map_equal_div_expr_except_constant(
	__isl_keep isl_basic_map *bmap1, int pos1,
	__isl_keep isl_basic_map *bmap2, int pos2);
__isl_give isl_basic_map *isl_basic_map_set_div_expr_constant_num_si_inplace(
	__isl_take isl_basic_map *bmap, int div, int value);

#endif
````
- **EN**: This block uses preprocessor directives to control compilation, macros, or include guards; declares or defines routines around `isl_basic_set_extract_equalities`, `isl_basic_map_equal_div_expr_part`, `isl_basic_map_equal_div_expr_except_constant`, `isl_basic_map_set_div_expr_constant_num_si_inplace`; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 使用预处理指令控制编译、宏或头文件保护; 声明或定义与 `isl_basic_set_extract_equalities`, `isl_basic_map_equal_div_expr_part`, `isl_basic_map_equal_div_expr_except_constant`, `isl_basic_map_set_div_expr_constant_num_si_inplace` 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

## Key Concepts / 关键概念

- **ISL relations and sets**
  - **CN**: ISL 关系与集合
- **Template/type wrappers**
  - **CN**: 模板与类型包装
- **Public interface declarations**
  - **CN**: 公共接口声明

## Dependencies / 依赖关系

- **ISL headers**: `isl/list.h`, `isl/set.h`, `isl/map.h`, `isl/vec.h`, `isl/hash.h` — ISL relation/set primitives that support polyhedral analysis and transformations.
  **ISL headers（CN）**：`isl/list.h`, `isl/set.h`, `isl/map.h`, `isl/vec.h`, `isl/hash.h` —— 支撑多面体分析与变换的 ISL 关系/集合原语。
- **System/standard headers**: `isl_reordering.h`, `isl_blk.h`, `isl_maybe_aff.h`, `isl_list_templ.h`, `isl_list_templ.h` — Standard-library or system declarations required by the implementation.
  **System/standard headers（CN）**：`isl_reordering.h`, `isl_blk.h`, `isl_maybe_aff.h`, `isl_list_templ.h`, `isl_list_templ.h` —— 实现所需的标准库或系统声明。
