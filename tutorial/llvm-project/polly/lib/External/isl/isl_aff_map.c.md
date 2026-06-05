# isl_aff_map.c — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `polly/lib/External/isl/isl_aff_map.c` | `polly/lib/External/isl/isl_aff_map.c` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | Implements or embeds upstream ISL support code used by Polly. | 实现或内嵌 Polly 使用的上游 ISL 支撑代码。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-22

````c
/*
 * Copyright 2011      INRIA Saclay
 * Copyright 2012-2013 Ecole Normale Superieure
 * Copyright 2016      Sven Verdoolaege
 *
 * Use of this software is governed by the MIT license
 *
 * Written by Sven Verdoolaege, INRIA Saclay - Ile-de-France,
 * Parc Club Orsay Universite, ZAC des vignes, 4 rue Jacques Monod,
 * 91893 Orsay, France
 * and Ecole Normale Superieure, 45 rue d'Ulm, 75230 Paris, France
 */

#include <isl/ctx.h>
#include <isl/space.h>
#include <isl/local_space.h>
#include <isl/union_map.h>
#include <isl_map_private.h>
#include <isl_aff_private.h>
#include <isl_vec_private.h>
#include <isl_seq.h>

````
- **EN**: This block imports ISL, system/standard headers needed by the surrounding code; uses ISL data structures or helpers for polyhedral reasoning; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 引入周边逻辑所需的 ISL、system/standard 头文件; 使用 ISL 数据结构或辅助函数进行多面体推理; 保留用于解释意图、用法或算法背景的注释.

### Lines 23-48

````c
#include <bset_from_bmap.c>
#include <set_from_map.c>

/* Check that the input living in "space" lives in a map space.
 * That is, check that "space" is a map space.
 */
static isl_stat check_input_is_map(__isl_keep isl_space *space)
{
	isl_bool is_set;

	is_set = isl_space_is_set(space);
	if (is_set < 0)
		return isl_stat_error;
	if (is_set)
		isl_die(isl_space_get_ctx(space), isl_error_invalid,
			"space of input is not a map", return isl_stat_error);
	return isl_stat_ok;
}

/* Check that the input living in "space" lives in a set space.
 * That is, check that "space" is a set space.
 */
static isl_stat check_input_is_set(__isl_keep isl_space *space)
{
	isl_bool is_set;

````
- **EN**: This block imports system/standard headers needed by the surrounding code; declares or defines routines around `check_input_is_map`, `isl_space_is_set`, `isl_die`, `check_input_is_set`; contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 引入周边逻辑所需的 system/standard 头文件; 声明或定义与 `check_input_is_map`, `isl_space_is_set`, `isl_die`, `check_input_is_set` 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 49-72

````c
	is_set = isl_space_is_set(space);
	if (is_set < 0)
		return isl_stat_error;
	if (!is_set)
		isl_die(isl_space_get_ctx(space), isl_error_invalid,
			"space of input is not a set", return isl_stat_error);
	return isl_stat_ok;
}

/* Construct a basic map mapping the domain of the affine expression
 * to a one-dimensional range prescribed by the affine expression.
 * If "rational" is set, then construct a rational basic map.
 *
 * A NaN affine expression cannot be converted to a basic map.
 */
static __isl_give isl_basic_map *isl_basic_map_from_aff2(
	__isl_take isl_aff *aff, int rational)
{
	int k;
	int pos;
	isl_bool is_nan;
	isl_local_space *ls;
	isl_basic_map *bmap = NULL;

````
- **EN**: This block declares or defines routines around `isl_space_is_set`, `isl_die`, `isl_basic_map_from_aff2`; contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_space_is_set`, `isl_die`, `isl_basic_map_from_aff2` 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 73-94

````c
	if (!aff)
		return NULL;
	is_nan = isl_aff_is_nan(aff);
	if (is_nan < 0)
		goto error;
	if (is_nan)
		isl_die(isl_aff_get_ctx(aff), isl_error_invalid,
			"cannot convert NaN", goto error);

	ls = isl_aff_get_local_space(aff);
	bmap = isl_basic_map_from_local_space(ls);
	bmap = isl_basic_map_extend_constraints(bmap, 1, 0);
	k = isl_basic_map_alloc_equality(bmap);
	if (k < 0)
		goto error;

	pos = isl_basic_map_offset(bmap, isl_dim_out);
	isl_seq_cpy(bmap->eq[k], aff->v->el + 1, pos);
	isl_int_neg(bmap->eq[k][pos], aff->v->el[0]);
	isl_seq_cpy(bmap->eq[k] + pos + 1, aff->v->el + 1 + pos,
		    aff->v->size - (pos + 1));

````
- **EN**: This block declares or defines routines around `isl_aff_is_nan`, `isl_die`, `isl_aff_get_local_space`, `isl_basic_map_from_local_space` (+5 more); contains control flow with 4 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_aff_is_nan`, `isl_die`, `isl_aff_get_local_space`, `isl_basic_map_from_local_space` (+5 more) 相关的例程; 包含控制流结构：4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 95-114

````c
	isl_aff_free(aff);
	if (rational)
		bmap = isl_basic_map_set_rational(bmap);
	bmap = isl_basic_map_gauss(bmap, NULL);
	bmap = isl_basic_map_finalize(bmap);
	return bmap;
error:
	isl_aff_free(aff);
	isl_basic_map_free(bmap);
	return NULL;
}

/* Construct a basic map mapping the domain of the affine expression
 * to a one-dimensional range prescribed by the affine expression.
 */
__isl_give isl_basic_map *isl_basic_map_from_aff(__isl_take isl_aff *aff)
{
	return isl_basic_map_from_aff2(aff, 0);
}

````
- **EN**: This block declares or defines routines around `isl_aff_free`, `isl_basic_map_set_rational`, `isl_basic_map_gauss`, `isl_basic_map_finalize` (+2 more); contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_aff_free`, `isl_basic_map_set_rational`, `isl_basic_map_gauss`, `isl_basic_map_finalize` (+2 more) 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 115-138

````c
/* Construct a map mapping the domain of the affine expression
 * to a one-dimensional range prescribed by the affine expression.
 */
__isl_give isl_map *isl_map_from_aff(__isl_take isl_aff *aff)
{
	isl_basic_map *bmap;

	bmap = isl_basic_map_from_aff(aff);
	return isl_map_from_basic_map(bmap);
}

/* Construct a basic map mapping the domain of the multi-affine expression
 * to its range, with each dimension in the range equated to the
 * corresponding affine expression.
 * If "rational" is set, then construct a rational basic map.
 */
__isl_give isl_basic_map *isl_basic_map_from_multi_aff2(
	__isl_take isl_multi_aff *maff, int rational)
{
	int i;
	isl_size dim;
	isl_space *space;
	isl_basic_map *bmap;

````
- **EN**: This block declares or defines routines around `isl_map_from_aff`, `isl_basic_map_from_aff`, `isl_basic_map_from_multi_aff2`; emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `isl_map_from_aff`, `isl_basic_map_from_aff`, `isl_basic_map_from_multi_aff2` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理; 保留用于解释意图、用法或算法背景的注释.

### Lines 139-158

````c
	dim = isl_multi_aff_dim(maff, isl_dim_out);
	if (dim < 0)
		goto error;

	if (dim != maff->n)
		isl_die(isl_multi_aff_get_ctx(maff), isl_error_internal,
			"invalid space", goto error);

	space = isl_space_domain(isl_multi_aff_get_space(maff));
	bmap = isl_basic_map_universe(isl_space_from_domain(space));
	if (rational)
		bmap = isl_basic_map_set_rational(bmap);

	for (i = 0; i < maff->n; ++i) {
		isl_aff *aff;
		isl_basic_map *bmap_i;

		aff = isl_aff_copy(maff->u.p[i]);
		bmap_i = isl_basic_map_from_aff2(aff, rational);

````
- **EN**: This block declares or defines routines around `isl_multi_aff_dim`, `isl_die`, `isl_space_domain`, `isl_basic_map_universe` (+3 more); contains control flow with 1 loop construct(s), 3 conditional check(s); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_multi_aff_dim`, `isl_die`, `isl_space_domain`, `isl_basic_map_universe` (+3 more) 相关的例程; 包含控制流结构：1 处循环、3 处条件判断; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 159-181

````c
		bmap = isl_basic_map_flat_range_product(bmap, bmap_i);
	}

	bmap = isl_basic_map_reset_space(bmap, isl_multi_aff_get_space(maff));

	isl_multi_aff_free(maff);
	return bmap;
error:
	isl_multi_aff_free(maff);
	return NULL;
}

/* Construct a basic map mapping the domain of the multi-affine expression
 * to its range, with each dimension in the range equated to the
 * corresponding affine expression.
 * If "ma" lives in a set space, then the result is actually a set.
 */
static __isl_give isl_basic_map *basic_map_from_multi_aff(
	__isl_take isl_multi_aff *ma)
{
	return isl_basic_map_from_multi_aff2(ma, 0);
}

````
- **EN**: This block declares or defines routines around `isl_basic_map_flat_range_product`, `isl_basic_map_reset_space`, `isl_multi_aff_free`, `basic_map_from_multi_aff`; emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `isl_basic_map_flat_range_product`, `isl_basic_map_reset_space`, `isl_multi_aff_free`, `basic_map_from_multi_aff` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理; 保留用于解释意图、用法或算法背景的注释.

### Lines 182-205

````c
/* Construct a basic map mapping the domain of the multi-affine expression
 * to its range, with each dimension in the range equated to the
 * corresponding affine expression.
 */
__isl_give isl_basic_map *isl_basic_map_from_multi_aff(
	__isl_take isl_multi_aff *ma)
{
	if (check_input_is_map(isl_multi_aff_peek_space(ma)) < 0)
		ma = isl_multi_aff_free(ma);
	return basic_map_from_multi_aff(ma);
}

/* Construct a basic set mapping the parameter domain
 * of the multi-affine expression to its space, with each dimension
 * in the space equated to the corresponding affine expression.
 */
__isl_give isl_basic_set *isl_basic_set_from_multi_aff(
	__isl_take isl_multi_aff *ma)
{
	if (check_input_is_set(isl_multi_aff_peek_space(ma)) < 0)
		ma = isl_multi_aff_free(ma);
	return bset_from_bmap(basic_map_from_multi_aff(ma));
}

````
- **EN**: This block declares or defines routines around `isl_basic_map_from_multi_aff`, `isl_multi_aff_free`, `isl_basic_set_from_multi_aff`; contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_basic_map_from_multi_aff`, `isl_multi_aff_free`, `isl_basic_set_from_multi_aff` 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 206-230

````c
/* Construct a map mapping the domain of the multi-affine expression
 * to its range, with each dimension in the range equated to the
 * corresponding affine expression.
 * If "maff" lives in a set space, then the result is actually a set.
 */
__isl_give isl_map *isl_map_from_multi_aff_internal(
	__isl_take isl_multi_aff *maff)
{
	isl_basic_map *bmap;

	bmap = basic_map_from_multi_aff(maff);
	return isl_map_from_basic_map(bmap);
}

/* Construct a map mapping the domain the multi-affine expression
 * to its range, with each dimension in the range equated to the
 * corresponding affine expression.
 */
__isl_give isl_map *isl_map_from_multi_aff(__isl_take isl_multi_aff *ma)
{
	if (check_input_is_map(isl_multi_aff_peek_space(ma)) < 0)
		ma = isl_multi_aff_free(ma);
	return isl_map_from_multi_aff_internal(ma);
}

````
- **EN**: This block declares or defines routines around `isl_map_from_multi_aff_internal`, `basic_map_from_multi_aff`, `isl_map_from_multi_aff`, `isl_multi_aff_free`; contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_map_from_multi_aff_internal`, `basic_map_from_multi_aff`, `isl_map_from_multi_aff`, `isl_multi_aff_free` 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 231-257

````c
/* This function performs the same operation as isl_map_from_multi_aff,
 * but is considered as a function on an isl_multi_aff when exported.
 */
__isl_give isl_map *isl_multi_aff_as_map(__isl_take isl_multi_aff *ma)
{
	return isl_map_from_multi_aff(ma);
}

/* Construct a set mapping the parameter domain the multi-affine expression
 * to its space, with each dimension in the space equated to the
 * corresponding affine expression.
 */
__isl_give isl_set *isl_set_from_multi_aff(__isl_take isl_multi_aff *ma)
{
	if (check_input_is_set(isl_multi_aff_peek_space(ma)) < 0)
		ma = isl_multi_aff_free(ma);
	return isl_map_from_multi_aff_internal(ma);
}

/* This function performs the same operation as isl_set_from_multi_aff,
 * but is considered as a function on an isl_multi_aff when exported.
 */
__isl_give isl_set *isl_multi_aff_as_set(__isl_take isl_multi_aff *ma)
{
	return isl_set_from_multi_aff(ma);
}

````
- **EN**: This block declares or defines routines around `isl_multi_aff_as_map`, `isl_set_from_multi_aff`, `isl_multi_aff_free`, `isl_multi_aff_as_set`; contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_multi_aff_as_map`, `isl_set_from_multi_aff`, `isl_multi_aff_free`, `isl_multi_aff_as_set` 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 258-277

````c
/* Construct a basic map mapping a domain in the given space to
 * to an n-dimensional range, with n the number of elements in the list,
 * where each coordinate in the range is prescribed by the
 * corresponding affine expression.
 * The domains of all affine expressions in the list are assumed to match
 * domain_space.
 */
__isl_give isl_basic_map *isl_basic_map_from_aff_list(
	__isl_take isl_space *domain_space, __isl_take isl_aff_list *list)
{
	int i;
	isl_space *space;
	isl_basic_map *bmap;

	if (!list)
		return NULL;

	space = isl_space_from_domain(domain_space);
	bmap = isl_basic_map_universe(space);

````
- **EN**: This block declares or defines routines around `isl_basic_map_from_aff_list`, `isl_space_from_domain`, `isl_basic_map_universe`; contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_basic_map_from_aff_list`, `isl_space_from_domain`, `isl_basic_map_universe` 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 278-301

````c
	for (i = 0; i < list->n; ++i) {
		isl_aff *aff;
		isl_basic_map *bmap_i;

		aff = isl_aff_copy(list->p[i]);
		bmap_i = isl_basic_map_from_aff(aff);

		bmap = isl_basic_map_flat_range_product(bmap, bmap_i);
	}

	isl_aff_list_free(list);
	return bmap;
}

/* Construct a map with as domain the domain of pwaff and
 * one-dimensional range corresponding to the affine expressions.
 * If "pwaff" lives in a set space, then the result is actually a set.
 */
__isl_give isl_map *isl_map_from_pw_aff_internal(__isl_take isl_pw_aff *pwaff)
{
	int i;
	isl_space *space;
	isl_map *map;

````
- **EN**: This block declares or defines routines around `isl_aff_copy`, `isl_basic_map_from_aff`, `isl_basic_map_flat_range_product`, `isl_aff_list_free` (+1 more); contains control flow with 1 loop construct(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_aff_copy`, `isl_basic_map_from_aff`, `isl_basic_map_flat_range_product`, `isl_aff_list_free` (+1 more) 相关的例程; 包含控制流结构：1 处循环; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 302-323

````c
	if (!pwaff)
		return NULL;

	space = isl_pw_aff_get_space(pwaff);
	map = isl_map_empty(space);

	for (i = 0; i < pwaff->n; ++i) {
		isl_basic_map *bmap;
		isl_map *map_i;

		bmap = isl_basic_map_from_aff(isl_aff_copy(pwaff->p[i].aff));
		map_i = isl_map_from_basic_map(bmap);
		map_i = isl_map_intersect_domain(map_i,
						isl_set_copy(pwaff->p[i].set));
		map = isl_map_union_disjoint(map, map_i);
	}

	isl_pw_aff_free(pwaff);

	return map;
}

````
- **EN**: This block declares or defines routines around `isl_pw_aff_get_space`, `isl_map_empty`, `isl_basic_map_from_aff`, `isl_map_from_basic_map` (+4 more); contains control flow with 1 loop construct(s), 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_pw_aff_get_space`, `isl_map_empty`, `isl_basic_map_from_aff`, `isl_map_from_basic_map` (+4 more) 相关的例程; 包含控制流结构：1 处循环、1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 324-347

````c
/* Construct a map with as domain the domain of pwaff and
 * one-dimensional range corresponding to the affine expressions.
 */
__isl_give isl_map *isl_map_from_pw_aff(__isl_take isl_pw_aff *pwaff)
{
	if (check_input_is_map(isl_pw_aff_peek_space(pwaff)) < 0)
		pwaff = isl_pw_aff_free(pwaff);
	return isl_map_from_pw_aff_internal(pwaff);
}

/* This function performs the same operation as isl_map_from_pw_aff,
 * but is considered as a function on an isl_pw_aff when exported.
 */
__isl_give isl_map *isl_pw_aff_as_map(__isl_take isl_pw_aff *pa)
{
	return isl_map_from_pw_aff(pa);
}

/* Construct a one-dimensional set with as parameter domain
 * the domain of pwaff and the single set dimension
 * corresponding to the affine expressions.
 */
__isl_give isl_set *isl_set_from_pw_aff(__isl_take isl_pw_aff *pwaff)
{
````
- **EN**: This block declares or defines routines around `isl_map_from_pw_aff`, `isl_pw_aff_free`, `isl_pw_aff_as_map`, `isl_set_from_pw_aff`; contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_map_from_pw_aff`, `isl_pw_aff_free`, `isl_pw_aff_as_map`, `isl_set_from_pw_aff` 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 348-368

````c
	if (check_input_is_set(isl_pw_aff_peek_space(pwaff)) < 0)
		pwaff = isl_pw_aff_free(pwaff);
	return set_from_map(isl_map_from_pw_aff_internal(pwaff));
}

/* Construct a map mapping the domain of the piecewise multi-affine expression
 * to its range, with each dimension in the range equated to the
 * corresponding affine expression on its cell.
 * If "pma" lives in a set space, then the result is actually a set.
 *
 * If the domain of "pma" is rational, then so is the constructed "map".
 */
__isl_give isl_map *isl_map_from_pw_multi_aff_internal(
	__isl_take isl_pw_multi_aff *pma)
{
	int i;
	isl_map *map;

	if (!pma)
		return NULL;

````
- **EN**: This block declares or defines routines around `isl_pw_aff_free`, `isl_map_from_pw_multi_aff_internal`; contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_pw_aff_free`, `isl_map_from_pw_multi_aff_internal` 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 369-391

````c
	map = isl_map_empty(isl_pw_multi_aff_get_space(pma));

	for (i = 0; i < pma->n; ++i) {
		isl_bool rational;
		isl_multi_aff *maff;
		isl_basic_map *bmap;
		isl_map *map_i;

		rational = isl_set_is_rational(pma->p[i].set);
		if (rational < 0)
			map = isl_map_free(map);
		maff = isl_multi_aff_copy(pma->p[i].maff);
		bmap = isl_basic_map_from_multi_aff2(maff, rational);
		map_i = isl_map_from_basic_map(bmap);
		map_i = isl_map_intersect_domain(map_i,
						isl_set_copy(pma->p[i].set));
		map = isl_map_union_disjoint(map, map_i);
	}

	isl_pw_multi_aff_free(pma);
	return map;
}

````
- **EN**: This block declares or defines routines around `isl_map_empty`, `isl_set_is_rational`, `isl_map_free`, `isl_multi_aff_copy` (+6 more); contains control flow with 1 loop construct(s), 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_map_empty`, `isl_set_is_rational`, `isl_map_free`, `isl_multi_aff_copy` (+6 more) 相关的例程; 包含控制流结构：1 处循环、1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 392-417

````c
/* Construct a map mapping the domain of the piecewise multi-affine expression
 * to its range, with each dimension in the range equated to the
 * corresponding affine expression on its cell.
 */
__isl_give isl_map *isl_map_from_pw_multi_aff(__isl_take isl_pw_multi_aff *pma)
{
	if (check_input_is_map(isl_pw_multi_aff_peek_space(pma)) < 0)
		pma = isl_pw_multi_aff_free(pma);
	return isl_map_from_pw_multi_aff_internal(pma);
}

/* This function performs the same operation as isl_map_from_pw_multi_aff,
 * but is considered as a function on an isl_pw_multi_aff when exported.
 */
__isl_give isl_map *isl_pw_multi_aff_as_map(__isl_take isl_pw_multi_aff *pma)
{
	return isl_map_from_pw_multi_aff(pma);
}

__isl_give isl_set *isl_set_from_pw_multi_aff(__isl_take isl_pw_multi_aff *pma)
{
	if (check_input_is_set(isl_pw_multi_aff_peek_space(pma)) < 0)
		pma = isl_pw_multi_aff_free(pma);
	return set_from_map(isl_map_from_pw_multi_aff_internal(pma));
}

````
- **EN**: This block declares or defines routines around `isl_map_from_pw_multi_aff`, `isl_pw_multi_aff_free`, `isl_pw_multi_aff_as_map`, `isl_set_from_pw_multi_aff`; contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_map_from_pw_multi_aff`, `isl_pw_multi_aff_free`, `isl_pw_multi_aff_as_map`, `isl_set_from_pw_multi_aff` 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 418-441

````c
/* This function performs the same operation as isl_set_from_pw_multi_aff,
 * but is considered as a function on an isl_pw_multi_aff when exported.
 */
__isl_give isl_set *isl_pw_multi_aff_as_set(__isl_take isl_pw_multi_aff *pma)
{
	return isl_set_from_pw_multi_aff(pma);
}

/* Construct a set or map mapping the shared (parameter) domain
 * of the piecewise affine expressions to the range of "mpa"
 * with each dimension in the range equated to the
 * corresponding piecewise affine expression.
 *
 * If "mpa" has an explicit domain (i.e., it is zero-dimensional),
 * then return a set or map with the same (parameter) domain.
 */
static __isl_give isl_map *map_from_multi_pw_aff(
	__isl_take isl_multi_pw_aff *mpa)
{
	int i;
	isl_size dim;
	isl_space *space;
	isl_map *map;

````
- **EN**: This block declares or defines routines around `isl_pw_multi_aff_as_set`, `shared`, `domain`, `same` (+1 more); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `isl_pw_multi_aff_as_set`, `shared`, `domain`, `same` (+1 more) 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理; 保留用于解释意图、用法或算法背景的注释.

### Lines 442-462

````c
	dim = isl_multi_pw_aff_dim(mpa, isl_dim_out);
	if (dim < 0)
		goto error;

	if (isl_space_dim(mpa->space, isl_dim_out) != mpa->n)
		isl_die(isl_multi_pw_aff_get_ctx(mpa), isl_error_internal,
			"invalid space", goto error);

	space = isl_multi_pw_aff_get_domain_space(mpa);
	map = isl_map_universe(isl_space_from_domain(space));

	for (i = 0; i < mpa->n; ++i) {
		isl_pw_aff *pa;
		isl_map *map_i;

		pa = isl_pw_aff_copy(mpa->u.p[i]);
		map_i = isl_map_from_pw_aff_internal(pa);

		map = isl_map_flat_range_product(map, map_i);
	}

````
- **EN**: This block declares or defines routines around `isl_multi_pw_aff_dim`, `isl_die`, `isl_multi_pw_aff_get_domain_space`, `isl_map_universe` (+3 more); contains control flow with 1 loop construct(s), 2 conditional check(s); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_multi_pw_aff_dim`, `isl_die`, `isl_multi_pw_aff_get_domain_space`, `isl_map_universe` (+3 more) 相关的例程; 包含控制流结构：1 处循环、2 处条件判断; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 463-485

````c
	map = isl_map_reset_space(map, isl_multi_pw_aff_get_space(mpa));

	map = isl_map_intersect_multi_pw_aff_explicit_domain(map, mpa);

	isl_multi_pw_aff_free(mpa);
	return map;
error:
	isl_multi_pw_aff_free(mpa);
	return NULL;
}

/* Construct a map mapping the shared domain
 * of the piecewise affine expressions to the range of "mpa"
 * with each dimension in the range equated to the
 * corresponding piecewise affine expression.
 */
__isl_give isl_map *isl_map_from_multi_pw_aff(__isl_take isl_multi_pw_aff *mpa)
{
	if (check_input_is_map(isl_multi_pw_aff_peek_space(mpa)) < 0)
		mpa = isl_multi_pw_aff_free(mpa);
	return map_from_multi_pw_aff(mpa);
}

````
- **EN**: This block declares or defines routines around `isl_map_reset_space`, `isl_map_intersect_multi_pw_aff_explicit_domain`, `isl_multi_pw_aff_free`, `isl_map_from_multi_pw_aff`; contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_map_reset_space`, `isl_map_intersect_multi_pw_aff_explicit_domain`, `isl_multi_pw_aff_free`, `isl_map_from_multi_pw_aff` 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 486-505

````c
/* This function performs the same operation as isl_map_from_multi_pw_aff,
 * but is considered as a function on an isl_multi_pw_aff when exported.
 */
__isl_give isl_map *isl_multi_pw_aff_as_map(__isl_take isl_multi_pw_aff *mpa)
{
	return isl_map_from_multi_pw_aff(mpa);
}

/* Construct a set mapping the shared parameter domain
 * of the piecewise affine expressions to the space of "mpa"
 * with each dimension in the range equated to the
 * corresponding piecewise affine expression.
 */
__isl_give isl_set *isl_set_from_multi_pw_aff(__isl_take isl_multi_pw_aff *mpa)
{
	if (check_input_is_set(isl_multi_pw_aff_peek_space(mpa)) < 0)
		mpa = isl_multi_pw_aff_free(mpa);
	return set_from_map(map_from_multi_pw_aff(mpa));
}

````
- **EN**: This block declares or defines routines around `isl_multi_pw_aff_as_map`, `isl_set_from_multi_pw_aff`, `isl_multi_pw_aff_free`; contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_multi_pw_aff_as_map`, `isl_set_from_multi_pw_aff`, `isl_multi_pw_aff_free` 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 506-526

````c
/* This function performs the same operation as isl_set_from_multi_pw_aff,
 * but is considered as a function on an isl_multi_pw_aff when exported.
 */
__isl_give isl_set *isl_multi_pw_aff_as_set(__isl_take isl_multi_pw_aff *mpa)
{
	return isl_set_from_multi_pw_aff(mpa);
}

/* Convert "pa" to an isl_map and add it to *umap.
 */
static isl_stat map_from_pw_aff_entry(__isl_take isl_pw_aff *pa, void *user)
{
	isl_union_map **umap = user;
	isl_map *map;

	map = isl_map_from_pw_aff(pa);
	*umap = isl_union_map_add_map(*umap, map);

	return *umap ? isl_stat_ok : isl_stat_error;
}

````
- **EN**: This block declares or defines routines around `isl_multi_pw_aff_as_set`, `map_from_pw_aff_entry`, `isl_map_from_pw_aff`, `isl_union_map_add_map`; emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `isl_multi_pw_aff_as_set`, `map_from_pw_aff_entry`, `isl_map_from_pw_aff`, `isl_union_map_add_map` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理; 保留用于解释意图、用法或算法背景的注释.

### Lines 527-546

````c
/* Construct a union map mapping the domain of the union
 * piecewise affine expression to its range, with the single output dimension
 * equated to the corresponding affine expressions on their cells.
 */
__isl_give isl_union_map *isl_union_map_from_union_pw_aff(
	__isl_take isl_union_pw_aff *upa)
{
	isl_space *space;
	isl_union_map *umap;

	if (!upa)
		return NULL;

	space = isl_union_pw_aff_get_space(upa);
	umap = isl_union_map_empty(space);

	if (isl_union_pw_aff_foreach_pw_aff(upa, &map_from_pw_aff_entry,
						&umap) < 0)
		umap = isl_union_map_free(umap);

````
- **EN**: This block declares or defines routines around `isl_union_map_from_union_pw_aff`, `isl_union_pw_aff_get_space`, `isl_union_map_empty`, `isl_union_map_free`; contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_union_map_from_union_pw_aff`, `isl_union_pw_aff_get_space`, `isl_union_map_empty`, `isl_union_map_free` 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 547-574

````c
	isl_union_pw_aff_free(upa);
	return umap;
}

/* Convert "pma" to an isl_map and add it to *umap.
 */
static isl_stat map_from_pw_multi_aff(__isl_take isl_pw_multi_aff *pma,
	void *user)
{
	isl_union_map **umap = user;
	isl_map *map;

	map = isl_map_from_pw_multi_aff(pma);
	*umap = isl_union_map_add_map(*umap, map);

	return isl_stat_ok;
}

/* Construct a union map mapping the domain of the union
 * piecewise multi-affine expression to its range, with each dimension
 * in the range equated to the corresponding affine expression on its cell.
 */
__isl_give isl_union_map *isl_union_map_from_union_pw_multi_aff(
	__isl_take isl_union_pw_multi_aff *upma)
{
	isl_space *space;
	isl_union_map *umap;

````
- **EN**: This block declares or defines routines around `isl_union_pw_aff_free`, `map_from_pw_multi_aff`, `isl_map_from_pw_multi_aff`, `isl_union_map_add_map` (+1 more); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `isl_union_pw_aff_free`, `map_from_pw_multi_aff`, `isl_map_from_pw_multi_aff`, `isl_union_map_add_map` (+1 more) 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理; 保留用于解释意图、用法或算法背景的注释.

### Lines 575-598

````c
	if (!upma)
		return NULL;

	space = isl_union_pw_multi_aff_get_space(upma);
	umap = isl_union_map_empty(space);

	if (isl_union_pw_multi_aff_foreach_pw_multi_aff(upma,
					&map_from_pw_multi_aff, &umap) < 0)
		goto error;

	isl_union_pw_multi_aff_free(upma);
	return umap;
error:
	isl_union_pw_multi_aff_free(upma);
	isl_union_map_free(umap);
	return NULL;
}

/* This function performs the same operation as
 * isl_union_map_from_union_pw_multi_aff,
 * but is considered as a function on an isl_union_pw_multi_aff when exported.
 */
__isl_give isl_union_map *isl_union_pw_multi_aff_as_union_map(
	__isl_take isl_union_pw_multi_aff *upma)
````
- **EN**: This block declares or defines routines around `isl_union_pw_multi_aff_get_space`, `isl_union_map_empty`, `isl_union_pw_multi_aff_free`, `isl_union_map_free` (+1 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_union_pw_multi_aff_get_space`, `isl_union_map_empty`, `isl_union_pw_multi_aff_free`, `isl_union_map_free` (+1 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 599-601

````c
{
	return isl_union_map_from_union_pw_multi_aff(upma);
}
````
- **EN**: This block emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

## Key Concepts / 关键概念

- **ISL relations and sets**
  - **CN**: ISL 关系与集合
- **Template/type wrappers**
  - **CN**: 模板与类型包装
- **Loop transformation**
  - **CN**: 循环变换

## Dependencies / 依赖关系

- **ISL headers**: `isl/ctx.h`, `isl/space.h`, `isl/local_space.h`, `isl/union_map.h` — ISL relation/set primitives that support polyhedral analysis and transformations.
  **ISL headers（CN）**：`isl/ctx.h`, `isl/space.h`, `isl/local_space.h`, `isl/union_map.h` —— 支撑多面体分析与变换的 ISL 关系/集合原语。
- **System/standard headers**: `isl_map_private.h`, `isl_aff_private.h`, `isl_vec_private.h`, `isl_seq.h`, `bset_from_bmap.c`, `set_from_map.c` — Standard-library or system declarations required by the implementation.
  **System/standard headers（CN）**：`isl_map_private.h`, `isl_aff_private.h`, `isl_vec_private.h`, `isl_seq.h`, `bset_from_bmap.c`, `set_from_map.c` —— 实现所需的标准库或系统声明。
