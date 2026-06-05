# isl_map_subtract.c — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `polly/lib/External/isl/isl_map_subtract.c` | `polly/lib/External/isl/isl_map_subtract.c` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | Implements or embeds upstream ISL support code used by Polly. | 实现或内嵌 Polly 使用的上游 ISL 支撑代码。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````c
/*
 * Copyright 2008-2009 Katholieke Universiteit Leuven
 *
 * Use of this software is governed by the MIT license
 *
 * Written by Sven Verdoolaege, K.U.Leuven, Departement
 * Computerwetenschappen, Celestijnenlaan 200A, B-3001 Leuven, Belgium
 */

#include <isl_map_private.h>
#include <isl_seq.h>
#include <isl/set.h>
#include <isl/map.h>
#include "isl_tab.h"
#include <isl_point_private.h>
#include <isl_vec_private.h>

#include <set_to_map.c>
#include <set_from_map.c>

````
- **EN**: This block imports ISL, system/standard headers needed by the surrounding code; uses ISL data structures or helpers for polyhedral reasoning; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 引入周边逻辑所需的 ISL、system/standard 头文件; 使用 ISL 数据结构或辅助函数进行多面体推理; 保留用于解释意图、用法或算法背景的注释.

### Lines 21-45

````c
/* Expand the constraint "c" into "v".  The initial "dim" dimensions
 * are the same, but "v" may have more divs than "c" and the divs of "c"
 * may appear in different positions in "v".
 * The number of divs in "c" is given by "n_div" and the mapping
 * of divs in "c" to divs in "v" is given by "div_map".
 *
 * Although it shouldn't happen in practice, it is theoretically
 * possible that two or more divs in "c" are mapped to the same div in "v".
 * These divs are then necessarily the same, so we simply add their
 * coefficients.
 */
static void expand_constraint(isl_vec *v, unsigned dim,
	isl_int *c, int *div_map, unsigned n_div)
{
	int i;

	isl_seq_cpy(v->el, c, 1 + dim);
	isl_seq_clr(v->el + 1 + dim, v->size - (1 + dim));

	for (i = 0; i < n_div; ++i) {
		int pos = 1 + dim + div_map[i];
		isl_int_add(v->el[pos], v->el[pos], c[1 + dim + i]);
	}
}

````
- **EN**: This block declares or defines routines around `expand_constraint`, `isl_seq_cpy`, `isl_seq_clr`, `isl_int_add`; contains control flow with 1 loop construct(s); uses ISL data structures or helpers for polyhedral reasoning; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `expand_constraint`, `isl_seq_cpy`, `isl_seq_clr`, `isl_int_add` 相关的例程; 包含控制流结构：1 处循环; 使用 ISL 数据结构或辅助函数进行多面体推理; 保留用于解释意图、用法或算法背景的注释.

### Lines 46-68

````c
/* Add all constraints of bmap to tab.  The equalities of bmap
 * are added as a pair of inequalities.
 */
static isl_stat tab_add_constraints(struct isl_tab *tab,
	__isl_keep isl_basic_map *bmap, int *div_map)
{
	int i;
	unsigned dim;
	isl_size tab_total;
	isl_size bmap_n_div;
	isl_size bmap_total;
	isl_vec *v;

	if (!tab || !bmap)
		return isl_stat_error;

	tab_total = isl_basic_map_dim(tab->bmap, isl_dim_all);
	bmap_total = isl_basic_map_dim(bmap, isl_dim_all);
	bmap_n_div = isl_basic_map_dim(bmap, isl_dim_div);
	dim = bmap_total - bmap_n_div;
	if (tab_total < 0 || bmap_total < 0 || bmap_n_div < 0)
		return isl_stat_error;

````
- **EN**: This block declares or defines routines around `tab_add_constraints`, `isl_basic_map_dim`; contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `tab_add_constraints`, `isl_basic_map_dim` 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 69-88

````c
	if (isl_tab_extend_cons(tab, 2 * bmap->n_eq + bmap->n_ineq) < 0)
		return isl_stat_error;

	v = isl_vec_alloc(bmap->ctx, 1 + tab_total);
	if (!v)
		return isl_stat_error;

	for (i = 0; i < bmap->n_eq; ++i) {
		expand_constraint(v, dim, bmap->eq[i], div_map, bmap_n_div);
		if (isl_tab_add_ineq(tab, v->el) < 0)
			goto error;
		isl_seq_neg(bmap->eq[i], bmap->eq[i], 1 + bmap_total);
		expand_constraint(v, dim, bmap->eq[i], div_map, bmap_n_div);
		if (isl_tab_add_ineq(tab, v->el) < 0)
			goto error;
		isl_seq_neg(bmap->eq[i], bmap->eq[i], 1 + bmap_total);
		if (tab->empty)
			break;
	}

````
- **EN**: This block declares or defines routines around `isl_vec_alloc`, `expand_constraint`, `isl_seq_neg`; contains control flow with 1 loop construct(s), 5 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_vec_alloc`, `expand_constraint`, `isl_seq_neg` 相关的例程; 包含控制流结构：1 处循环、5 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 89-112

````c
	for (i = 0; i < bmap->n_ineq; ++i) {
		expand_constraint(v, dim, bmap->ineq[i], div_map, bmap_n_div);
		if (isl_tab_add_ineq(tab, v->el) < 0)
			goto error;
		if (tab->empty)
			break;
	}

	isl_vec_free(v);
	return isl_stat_ok;
error:
	isl_vec_free(v);
	return isl_stat_error;
}

/* Add a specific constraint of bmap (or its opposite) to tab.
 * The position of the constraint is specified by "c", where
 * the equalities of bmap are counted twice, once for the inequality
 * that is equal to the equality, and once for its negation.
 *
 * Each of these constraints has been added to "tab" before by
 * tab_add_constraints (and later removed again), so there should
 * already be a row available for the constraint.
 */
````
- **EN**: This block declares or defines routines around `expand_constraint`, `isl_vec_free`, `bmap`, `tab_add_constraints`; contains control flow with 1 loop construct(s), 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `expand_constraint`, `isl_vec_free`, `bmap`, `tab_add_constraints` 相关的例程; 包含控制流结构：1 处循环、2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 113-132

````c
static isl_stat tab_add_constraint(struct isl_tab *tab,
	__isl_keep isl_basic_map *bmap, int *div_map, int c, int oppose)
{
	unsigned dim;
	isl_size tab_total;
	isl_size bmap_n_div;
	isl_size bmap_total;
	isl_vec *v;
	isl_stat r;

	if (!tab || !bmap)
		return isl_stat_error;

	tab_total = isl_basic_map_dim(tab->bmap, isl_dim_all);
	bmap_total = isl_basic_map_dim(bmap, isl_dim_all);
	bmap_n_div = isl_basic_map_dim(bmap, isl_dim_div);
	dim = bmap_total - bmap_n_div;
	if (tab_total < 0 || bmap_total < 0 || bmap_n_div < 0)
		return isl_stat_error;

````
- **EN**: This block declares or defines routines around `tab_add_constraint`, `isl_basic_map_dim`; contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `tab_add_constraint`, `isl_basic_map_dim` 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 133-156

````c
	v = isl_vec_alloc(bmap->ctx, 1 + tab_total);
	if (!v)
		return isl_stat_error;

	if (c < 2 * bmap->n_eq) {
		if ((c % 2) != oppose)
			isl_seq_neg(bmap->eq[c/2], bmap->eq[c/2],
					1 + bmap_total);
		if (oppose)
			isl_int_sub_ui(bmap->eq[c/2][0], bmap->eq[c/2][0], 1);
		expand_constraint(v, dim, bmap->eq[c/2], div_map, bmap_n_div);
		r = isl_tab_add_ineq(tab, v->el);
		if (oppose)
			isl_int_add_ui(bmap->eq[c/2][0], bmap->eq[c/2][0], 1);
		if ((c % 2) != oppose)
			isl_seq_neg(bmap->eq[c/2], bmap->eq[c/2],
					1 + bmap_total);
	} else {
		c -= 2 * bmap->n_eq;
		if (oppose) {
			isl_seq_neg(bmap->ineq[c], bmap->ineq[c],
					1 + bmap_total);
			isl_int_sub_ui(bmap->ineq[c][0], bmap->ineq[c][0], 1);
		}
````
- **EN**: This block declares or defines routines around `isl_vec_alloc`, `isl_seq_neg`, `isl_int_sub_ui`, `expand_constraint` (+2 more); contains control flow with 7 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_vec_alloc`, `isl_seq_neg`, `isl_int_sub_ui`, `expand_constraint` (+2 more) 相关的例程; 包含控制流结构：7 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 157-177

````c
		expand_constraint(v, dim, bmap->ineq[c], div_map, bmap_n_div);
		r = isl_tab_add_ineq(tab, v->el);
		if (oppose) {
			isl_int_add_ui(bmap->ineq[c][0], bmap->ineq[c][0], 1);
			isl_seq_neg(bmap->ineq[c], bmap->ineq[c],
					1 + bmap_total);
		}
	}

	isl_vec_free(v);
	return r;
}

static isl_stat tab_add_divs(struct isl_tab *tab,
	__isl_keep isl_basic_map *bmap, int **div_map)
{
	int i, j;
	struct isl_vec *vec;
	isl_size total;
	unsigned dim;

````
- **EN**: This block declares or references types such as `isl_vec`; declares or defines routines around `expand_constraint`, `isl_tab_add_ineq`, `isl_int_add_ui`, `isl_seq_neg` (+2 more); contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_vec`; 声明或定义与 `expand_constraint`, `isl_tab_add_ineq`, `isl_int_add_ui`, `isl_seq_neg` (+2 more) 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 178-201

````c
	if (!bmap)
		return isl_stat_error;
	if (!bmap->n_div)
		return isl_stat_ok;

	if (!*div_map)
		*div_map = isl_alloc_array(bmap->ctx, int, bmap->n_div);
	if (!*div_map)
		return isl_stat_error;

	total = isl_basic_map_dim(tab->bmap, isl_dim_all);
	if (total < 0)
		return isl_stat_error;
	dim = total - tab->bmap->n_div;
	vec = isl_vec_alloc(bmap->ctx, 2 + total + bmap->n_div);
	if (!vec)
		return isl_stat_error;

	for (i = 0; i < bmap->n_div; ++i) {
		isl_seq_cpy(vec->el, bmap->div[i], 2 + dim);
		isl_seq_clr(vec->el + 2 + dim, tab->bmap->n_div);
		for (j = 0; j < i; ++j)
			isl_int_add(vec->el[2 + dim + (*div_map)[j]],
				    vec->el[2 + dim + (*div_map)[j]],
````
- **EN**: This block declares or defines routines around `isl_alloc_array`, `isl_basic_map_dim`, `isl_vec_alloc`, `isl_seq_cpy` (+2 more); contains control flow with 2 loop construct(s), 6 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_alloc_array`, `isl_basic_map_dim`, `isl_vec_alloc`, `isl_seq_cpy` (+2 more) 相关的例程; 包含控制流结构：2 处循环、6 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 202-223

````c
					bmap->div[i][2 + dim + j]);
		for (j = 0; j < tab->bmap->n_div; ++j)
			if (isl_seq_eq(tab->bmap->div[j],
					vec->el, 2 + dim + tab->bmap->n_div))
				break;
		(*div_map)[i] = j;
		if (j == tab->bmap->n_div) {
			vec->size = 2 + dim + tab->bmap->n_div;
			if (isl_tab_add_div(tab, vec) < 0)
				goto error;
		}
	}

	isl_vec_free(vec);

	return isl_stat_ok;
error:
	isl_vec_free(vec);

	return isl_stat_error;
}

````
- **EN**: This block declares or defines routines around `isl_vec_free`; contains control flow with 1 loop construct(s), 3 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_vec_free` 相关的例程; 包含控制流结构：1 处循环、3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 224-246

````c
/* Freeze all constraints of tableau tab.
 */
static int tab_freeze_constraints(struct isl_tab *tab)
{
	int i;

	for (i = 0; i < tab->n_con; ++i)
		if (isl_tab_freeze_constraint(tab, i) < 0)
			return -1;

	return 0;
}

/* Check for redundant constraints starting at offset.
 * Put the indices of the non-redundant constraints in index
 * and return the number of non-redundant constraints.
 */
static int n_non_redundant(isl_ctx *ctx, struct isl_tab *tab,
	int offset, int **index)
{
	int i, n;
	int n_test = tab->n_con - offset;

````
- **EN**: This block declares or defines routines around `tab_freeze_constraints`, `n_non_redundant`; contains control flow with 1 loop construct(s), 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `tab_freeze_constraints`, `n_non_redundant` 相关的例程; 包含控制流结构：1 处循环、1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 247-266

````c
	if (isl_tab_detect_redundant(tab) < 0)
		return -1;

	if (n_test == 0)
		return 0;
	if (!*index)
		*index = isl_alloc_array(ctx, int, n_test);
	if (!*index)
		return -1;

	for (n = 0, i = 0; i < n_test; ++i) {
		int r;
		r = isl_tab_is_redundant(tab, offset + i);
		if (r < 0)
			return -1;
		if (r)
			continue;
		(*index)[n++] = i;
	}

````
- **EN**: This block declares or defines routines around `isl_alloc_array`, `isl_tab_is_redundant`; contains control flow with 1 loop construct(s), 6 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_alloc_array`, `isl_tab_is_redundant` 相关的例程; 包含控制流结构：1 处循环、6 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 267-290

````c
	return n;
}

/* basic_map_collect_diff calls add on each of the pieces of
 * the set difference between bmap and map until the add method
 * return a negative value.
 */
struct isl_diff_collector {
	isl_stat (*add)(struct isl_diff_collector *dc,
		    __isl_take isl_basic_map *bmap);
};

/* Compute the set difference between bmap and map and call
 * dc->add on each of the piece until this function returns
 * a negative value.
 * Return 0 on success and -1 on error.  dc->add returning
 * a negative value is treated as an error, but the calling
 * function can interpret the results based on the state of dc.
 *
 * Assumes that map has known divs.
 *
 * The difference is computed by a backtracking algorithm.
 * Each level corresponds to a basic map in "map".
 * When a node in entered for the first time, we check
````
- **EN**: This block declares or references types such as `isl_diff_collector`; declares or defines routines around `isl_stat`; emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_diff_collector`; 声明或定义与 `isl_stat` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 291-314

````c
 * if the corresonding basic map intersects the current piece
 * of "bmap".  If not, we move to the next level.
 * Otherwise, we split the current piece into as many
 * pieces as there are non-redundant constraints of the current
 * basic map in the intersection.  Each of these pieces is
 * handled by a child of the current node.
 * In particular, if there are n non-redundant constraints,
 * then for each 0 <= i < n, a piece is cut off by adding
 * constraints 0 <= j < i and adding the opposite of constraint i.
 * If there are no non-redundant constraints, meaning that the current
 * piece is a subset of the current basic map, then we simply backtrack.
 *
 * In the leaves, we check if the remaining piece has any integer points
 * and if so, pass it along to dc->add.  As a special case, if nothing
 * has been removed when we end up in a leaf, we simply pass along
 * the original basic map.
 */
static isl_stat basic_map_collect_diff(__isl_take isl_basic_map *bmap,
	__isl_take isl_map *map, struct isl_diff_collector *dc)
{
	int i;
	int modified;
	int level;
	int init;
````
- **EN**: This block declares or defines routines around `basic_map_collect_diff`; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `basic_map_collect_diff` 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 315-336

````c
	isl_bool empty;
	isl_ctx *ctx;
	struct isl_tab *tab = NULL;
	struct isl_tab_undo **snap = NULL;
	int *k = NULL;
	int *n = NULL;
	int **index = NULL;
	int **div_map = NULL;

	empty = isl_basic_map_is_empty(bmap);
	if (empty) {
		isl_basic_map_free(bmap);
		isl_map_free(map);
		return empty < 0 ? isl_stat_error : isl_stat_ok;
	}

	bmap = isl_basic_map_cow(bmap);
	map = isl_map_cow(map);

	if (!bmap || !map)
		goto error;

````
- **EN**: This block declares or references types such as `isl_tab`, `isl_tab_undo`; declares or defines routines around `isl_basic_map_is_empty`, `isl_basic_map_free`, `isl_map_free`, `isl_basic_map_cow` (+1 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_tab`, `isl_tab_undo`; 声明或定义与 `isl_basic_map_is_empty`, `isl_basic_map_free`, `isl_map_free`, `isl_basic_map_cow` (+1 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 337-356

````c
	ctx = map->ctx;
	snap = isl_alloc_array(map->ctx, struct isl_tab_undo *, map->n);
	k = isl_alloc_array(map->ctx, int, map->n);
	n = isl_alloc_array(map->ctx, int, map->n);
	index = isl_calloc_array(map->ctx, int *, map->n);
	div_map = isl_calloc_array(map->ctx, int *, map->n);
	if (!snap || !k || !n || !index || !div_map)
		goto error;

	bmap = isl_basic_map_order_divs(bmap);
	map = isl_map_order_divs(map);

	tab = isl_tab_from_basic_map(bmap, 1);
	if (!tab)
		goto error;

	modified = 0;
	level = 0;
	init = 1;

````
- **EN**: This block declares or defines routines around `isl_alloc_array`, `isl_calloc_array`, `isl_basic_map_order_divs`, `isl_map_order_divs` (+1 more); contains control flow with 2 conditional check(s); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_alloc_array`, `isl_calloc_array`, `isl_basic_map_order_divs`, `isl_map_order_divs` (+1 more) 相关的例程; 包含控制流结构：2 处条件判断; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 357-380

````c
	while (level >= 0) {
		if (level >= map->n) {
			int empty;
			struct isl_basic_map *bm;
			if (!modified) {
				if (dc->add(dc, isl_basic_map_copy(bmap)) < 0)
					goto error;
				break;
			}
			bm = isl_basic_map_copy(tab->bmap);
			bm = isl_basic_map_cow(bm);
			bm = isl_basic_map_update_from_tab(bm, tab);
			bm = isl_basic_map_simplify(bm);
			bm = isl_basic_map_finalize(bm);
			empty = isl_basic_map_is_empty(bm);
			if (empty)
				isl_basic_map_free(bm);
			else if (dc->add(dc, bm) < 0)
				goto error;
			if (empty < 0)
				goto error;
			level--;
			init = 0;
			continue;
````
- **EN**: This block declares or references types such as `isl_basic_map`; declares or defines routines around `isl_basic_map_copy`, `isl_basic_map_cow`, `isl_basic_map_update_from_tab`, `isl_basic_map_simplify` (+3 more); contains control flow with 1 loop construct(s), 6 conditional check(s); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或引用类型，例如 `isl_basic_map`; 声明或定义与 `isl_basic_map_copy`, `isl_basic_map_cow`, `isl_basic_map_update_from_tab`, `isl_basic_map_simplify` (+3 more) 相关的例程; 包含控制流结构：1 处循环、6 处条件判断; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 381-404

````c
		}
		if (init) {
			int offset;
			struct isl_tab_undo *snap2;
			snap2 = isl_tab_snap(tab);
			if (tab_add_divs(tab, map->p[level],
					 &div_map[level]) < 0)
				goto error;
			offset = tab->n_con;
			snap[level] = isl_tab_snap(tab);
			if (tab_freeze_constraints(tab) < 0)
				goto error;
			if (tab_add_constraints(tab, map->p[level],
						div_map[level]) < 0)
				goto error;
			k[level] = 0;
			n[level] = 0;
			if (tab->empty) {
				if (isl_tab_rollback(tab, snap2) < 0)
					goto error;
				level++;
				continue;
			}
			modified = 1;
````
- **EN**: This block declares or references types such as `isl_tab_undo`; declares or defines routines around `isl_tab_snap`; contains control flow with 6 conditional check(s); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或引用类型，例如 `isl_tab_undo`; 声明或定义与 `isl_tab_snap` 相关的例程; 包含控制流结构：6 处条件判断; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 405-428

````c
			n[level] = n_non_redundant(ctx, tab, offset,
						    &index[level]);
			if (n[level] < 0)
				goto error;
			if (n[level] == 0) {
				level--;
				init = 0;
				continue;
			}
			if (isl_tab_rollback(tab, snap[level]) < 0)
				goto error;
			if (tab_add_constraint(tab, map->p[level],
					div_map[level], index[level][0], 1) < 0)
				goto error;
			level++;
			continue;
		} else {
			if (k[level] + 1 >= n[level]) {
				level--;
				continue;
			}
			if (isl_tab_rollback(tab, snap[level]) < 0)
				goto error;
			if (tab_add_constraint(tab, map->p[level],
````
- **EN**: This block declares or defines routines around `n_non_redundant`; contains control flow with 7 conditional check(s); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `n_non_redundant` 相关的例程; 包含控制流结构：7 处条件判断; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 429-454

````c
						div_map[level],
						index[level][k[level]], 0) < 0)
				goto error;
			snap[level] = isl_tab_snap(tab);
			k[level]++;
			if (tab_add_constraint(tab, map->p[level],
						div_map[level],
						index[level][k[level]], 1) < 0)
				goto error;
			level++;
			init = 1;
			continue;
		}
	}

	isl_tab_free(tab);
	free(snap);
	free(n);
	free(k);
	for (i = 0; index && i < map->n; ++i)
		free(index[i]);
	free(index);
	for (i = 0; div_map && i < map->n; ++i)
		free(div_map[i]);
	free(div_map);

````
- **EN**: This block declares or defines routines around `isl_tab_snap`, `isl_tab_free`, `free`; contains control flow with 2 loop construct(s), 1 conditional check(s); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_tab_snap`, `isl_tab_free`, `free` 相关的例程; 包含控制流结构：2 处循环、1 处条件判断; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 455-474

````c
	isl_basic_map_free(bmap);
	isl_map_free(map);

	return isl_stat_ok;
error:
	isl_tab_free(tab);
	free(snap);
	free(n);
	free(k);
	for (i = 0; index && i < map->n; ++i)
		free(index[i]);
	free(index);
	for (i = 0; div_map && i < map->n; ++i)
		free(div_map[i]);
	free(div_map);
	isl_basic_map_free(bmap);
	isl_map_free(map);
	return isl_stat_error;
}

````
- **EN**: This block declares or defines routines around `isl_basic_map_free`, `isl_map_free`, `isl_tab_free`, `free`; contains control flow with 2 loop construct(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_basic_map_free`, `isl_map_free`, `isl_tab_free`, `free` 相关的例程; 包含控制流结构：2 处循环; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 475-496

````c
/* A diff collector that actually collects all parts of the
 * set difference in the field diff.
 */
struct isl_subtract_diff_collector {
	struct isl_diff_collector dc;
	struct isl_map *diff;
};

/* isl_subtract_diff_collector callback.
 */
static isl_stat basic_map_subtract_add(struct isl_diff_collector *dc,
			    __isl_take isl_basic_map *bmap)
{
	struct isl_subtract_diff_collector *sdc;
	sdc = (struct isl_subtract_diff_collector *)dc;

	sdc->diff = isl_map_union_disjoint(sdc->diff,
			isl_map_from_basic_map(bmap));

	return sdc->diff ? isl_stat_ok : isl_stat_error;
}

````
- **EN**: This block declares or references types such as `isl_subtract_diff_collector`, `isl_diff_collector`, `isl_map`; declares or defines routines around `basic_map_subtract_add`, `isl_map_union_disjoint`, `isl_map_from_basic_map`; emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_subtract_diff_collector`, `isl_diff_collector`, `isl_map`; 声明或定义与 `basic_map_subtract_add`, `isl_map_union_disjoint`, `isl_map_from_basic_map` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 497-518

````c
/* Return the set difference between bmap and map.
 */
static __isl_give isl_map *basic_map_subtract(__isl_take isl_basic_map *bmap,
	__isl_take isl_map *map)
{
	struct isl_subtract_diff_collector sdc;
	sdc.dc.add = &basic_map_subtract_add;
	sdc.diff = isl_map_empty(isl_basic_map_get_space(bmap));
	if (basic_map_collect_diff(bmap, map, &sdc.dc) < 0) {
		isl_map_free(sdc.diff);
		sdc.diff = NULL;
	}
	return sdc.diff;
}

/* Return an empty map living in the same space as "map1" and "map2".
 */
static __isl_give isl_map *replace_pair_by_empty( __isl_take isl_map *map1,
	__isl_take isl_map *map2)
{
	isl_space *space;

````
- **EN**: This block declares or references types such as `isl_subtract_diff_collector`; declares or defines routines around `basic_map_subtract`, `isl_map_empty`, `isl_map_free`, `replace_pair_by_empty`; contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_subtract_diff_collector`; 声明或定义与 `basic_map_subtract`, `isl_map_empty`, `isl_map_free`, `replace_pair_by_empty` 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 519-539

````c
	space = isl_map_get_space(map1);
	isl_map_free(map1);
	isl_map_free(map2);
	return isl_map_empty(space);
}

/* Return the set difference between map1 and map2.
 * (U_i A_i) \ (U_j B_j) is computed as U_i (A_i \ (U_j B_j))
 *
 * If "map1" and "map2" are obviously equal to each other,
 * then return an empty map in the same space.
 *
 * If "map1" and "map2" are disjoint, then simply return "map1".
 */
__isl_give isl_map *isl_map_subtract( __isl_take isl_map *map1,
	__isl_take isl_map *map2)
{
	int i;
	int equal, disjoint;
	struct isl_map *diff;

````
- **EN**: This block declares or references types such as `isl_map`; declares or defines routines around `isl_map_get_space`, `isl_map_free`, `U_i`, `isl_map_subtract`; emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_map`; 声明或定义与 `isl_map_get_space`, `isl_map_free`, `U_i`, `isl_map_subtract` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 540-563

````c
	if (isl_map_align_params_bin(&map1, &map2) < 0)
		goto error;
	if (isl_map_check_equal_space(map1, map2) < 0)
		goto error;

	equal = isl_map_plain_is_equal(map1, map2);
	if (equal < 0)
		goto error;
	if (equal)
		return replace_pair_by_empty(map1, map2);

	disjoint = isl_map_is_disjoint(map1, map2);
	if (disjoint < 0)
		goto error;
	if (disjoint) {
		isl_map_free(map2);
		return map1;
	}

	map1 = isl_map_compute_divs(map1);
	map2 = isl_map_compute_divs(map2);
	if (!map1 || !map2)
		goto error;

````
- **EN**: This block declares or defines routines around `isl_map_plain_is_equal`, `isl_map_is_disjoint`, `isl_map_free`, `isl_map_compute_divs`; contains control flow with 7 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_map_plain_is_equal`, `isl_map_is_disjoint`, `isl_map_free`, `isl_map_compute_divs` 相关的例程; 包含控制流结构：7 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 564-587

````c
	map1 = isl_map_remove_empty_parts(map1);
	map2 = isl_map_remove_empty_parts(map2);

	diff = isl_map_empty(isl_map_get_space(map1));
	for (i = 0; i < map1->n; ++i) {
		struct isl_map *d;
		d = basic_map_subtract(isl_basic_map_copy(map1->p[i]),
				       isl_map_copy(map2));
		if (ISL_F_ISSET(map1, ISL_MAP_DISJOINT))
			diff = isl_map_union_disjoint(diff, d);
		else
			diff = isl_map_union(diff, d);
	}

	isl_map_free(map1);
	isl_map_free(map2);

	return diff;
error:
	isl_map_free(map1);
	isl_map_free(map2);
	return NULL;
}

````
- **EN**: This block declares or references types such as `isl_map`; declares or defines routines around `isl_map_remove_empty_parts`, `isl_map_empty`, `basic_map_subtract`, `isl_map_copy` (+3 more); contains control flow with 1 loop construct(s), 1 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_map`; 声明或定义与 `isl_map_remove_empty_parts`, `isl_map_empty`, `basic_map_subtract`, `isl_map_copy` (+3 more) 相关的例程; 包含控制流结构：1 处循环、1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 588-610

````c
__isl_give isl_set *isl_set_subtract(__isl_take isl_set *set1,
	__isl_take isl_set *set2)
{
	return set_from_map(isl_map_subtract(set_to_map(set1),
					    set_to_map(set2)));
}

/* Remove the elements of "dom" from the domain of "map".
 */
__isl_give isl_map *isl_map_subtract_domain(__isl_take isl_map *map,
	__isl_take isl_set *dom)
{
	isl_bool ok;
	isl_map *ext_dom;

	isl_map_align_params_set(&map, &dom);
	ok = isl_map_compatible_domain(map, dom);
	if (ok < 0)
		goto error;
	if (!ok)
		isl_die(isl_set_get_ctx(dom), isl_error_invalid,
			"incompatible spaces", goto error);
	
````
- **EN**: This block declares or defines routines around `isl_set_subtract`, `set_to_map`, `isl_map_subtract_domain`, `isl_map_align_params_set` (+2 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_set_subtract`, `set_to_map`, `isl_map_subtract_domain`, `isl_map_align_params_set` (+2 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 611-635

````c
	ext_dom = isl_map_universe(isl_map_get_space(map));
	ext_dom = isl_map_intersect_domain(ext_dom, dom);
	return isl_map_subtract(map, ext_dom);
error:
	isl_map_free(map);
	isl_set_free(dom);
	return NULL;
}

/* Remove the elements of "dom" from the range of "map".
 */
__isl_give isl_map *isl_map_subtract_range(__isl_take isl_map *map,
	__isl_take isl_set *dom)
{
	isl_bool ok;
	isl_map *ext_dom;

	isl_map_align_params_set(&map, &dom);
	ok = isl_map_compatible_range(map, dom);
	if (ok < 0)
		goto error;
	if (!ok)
		isl_die(isl_set_get_ctx(dom), isl_error_invalid,
			"incompatible spaces", goto error);
	
````
- **EN**: This block declares or defines routines around `isl_map_universe`, `isl_map_intersect_domain`, `isl_map_free`, `isl_set_free` (+4 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_map_universe`, `isl_map_intersect_domain`, `isl_map_free`, `isl_set_free` (+4 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 636-660

````c
	ext_dom = isl_map_universe(isl_map_get_space(map));
	ext_dom = isl_map_intersect_range(ext_dom, dom);
	return isl_map_subtract(map, ext_dom);
error:
	isl_map_free(map);
	isl_set_free(dom);
	return NULL;
}

/* A diff collector that aborts as soon as its add function is called,
 * setting empty to isl_false.
 */
struct isl_is_empty_diff_collector {
	struct isl_diff_collector dc;
	isl_bool empty;
};

/* isl_is_empty_diff_collector callback.
 */
static isl_stat basic_map_is_empty_add(struct isl_diff_collector *dc,
			    __isl_take isl_basic_map *bmap)
{
	struct isl_is_empty_diff_collector *edc;
	edc = (struct isl_is_empty_diff_collector *)dc;

````
- **EN**: This block declares or references types such as `isl_is_empty_diff_collector`, `isl_diff_collector`; declares or defines routines around `isl_map_universe`, `isl_map_intersect_range`, `isl_map_free`, `isl_set_free` (+1 more); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_is_empty_diff_collector`, `isl_diff_collector`; 声明或定义与 `isl_map_universe`, `isl_map_intersect_range`, `isl_map_free`, `isl_set_free` (+1 more) 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 661-680

````c
	edc->empty = isl_bool_false;

	isl_basic_map_free(bmap);
	return isl_stat_error;
}

/* Check if bmap \ map is empty by computing this set difference
 * and breaking off as soon as the difference is known to be non-empty.
 */
static isl_bool basic_map_diff_is_empty(__isl_keep isl_basic_map *bmap,
	__isl_keep isl_map *map)
{
	isl_bool empty;
	isl_stat r;
	struct isl_is_empty_diff_collector edc;

	empty = isl_basic_map_plain_is_empty(bmap);
	if (empty)
		return empty;

````
- **EN**: This block declares or references types such as `isl_is_empty_diff_collector`; declares or defines routines around `isl_basic_map_free`, `basic_map_diff_is_empty`, `isl_basic_map_plain_is_empty`; contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_is_empty_diff_collector`; 声明或定义与 `isl_basic_map_free`, `basic_map_diff_is_empty`, `isl_basic_map_plain_is_empty` 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 681-702

````c
	edc.dc.add = &basic_map_is_empty_add;
	edc.empty = isl_bool_true;
	r = basic_map_collect_diff(isl_basic_map_copy(bmap),
				   isl_map_copy(map), &edc.dc);
	if (!edc.empty)
		return isl_bool_false;

	return r < 0 ? isl_bool_error : isl_bool_true;
}

/* Check if map1 \ map2 is empty by checking if the set difference is empty
 * for each of the basic maps in map1.
 */
static isl_bool map_diff_is_empty(__isl_keep isl_map *map1,
	__isl_keep isl_map *map2)
{
	int i;
	isl_bool is_empty = isl_bool_true;

	if (!map1 || !map2)
		return isl_bool_error;
	
````
- **EN**: This block declares or defines routines around `basic_map_collect_diff`, `isl_map_copy`, `map_diff_is_empty`; contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `basic_map_collect_diff`, `isl_map_copy`, `map_diff_is_empty` 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 703-729

````c
	for (i = 0; i < map1->n; ++i) {
		is_empty = basic_map_diff_is_empty(map1->p[i], map2);
		if (is_empty < 0 || !is_empty)
			 break;
	}

	return is_empty;
}

/* Return true if "bmap" contains a single element.
 */
isl_bool isl_basic_map_plain_is_singleton(__isl_keep isl_basic_map *bmap)
{
	isl_size total;

	if (!bmap)
		return isl_bool_error;
	if (bmap->n_div)
		return isl_bool_false;
	if (bmap->n_ineq)
		return isl_bool_false;
	total = isl_basic_map_dim(bmap, isl_dim_all);
	if (total < 0)
		return isl_bool_error;
	return bmap->n_eq == total;
}

````
- **EN**: This block declares or defines routines around `basic_map_diff_is_empty`, `isl_basic_map_plain_is_singleton`, `isl_basic_map_dim`; contains control flow with 1 loop construct(s), 5 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `basic_map_diff_is_empty`, `isl_basic_map_plain_is_singleton`, `isl_basic_map_dim` 相关的例程; 包含控制流结构：1 处循环、5 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 730-752

````c
/* Return true if "map" contains a single element.
 */
isl_bool isl_map_plain_is_singleton(__isl_keep isl_map *map)
{
	if (!map)
		return isl_bool_error;
	if (map->n != 1)
		return isl_bool_false;

	return isl_basic_map_plain_is_singleton(map->p[0]);
}

/* Given a singleton basic map, extract the single element
 * as an isl_point.
 */
static __isl_give isl_point *singleton_extract_point(
	__isl_keep isl_basic_map *bmap)
{
	int j;
	isl_size dim;
	struct isl_vec *point;
	isl_int m;

````
- **EN**: This block declares or references types such as `isl_vec`; declares or defines routines around `isl_map_plain_is_singleton`, `singleton_extract_point`; contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_vec`; 声明或定义与 `isl_map_plain_is_singleton`, `singleton_extract_point` 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 753-777

````c
	dim = isl_basic_map_dim(bmap, isl_dim_all);
	if (dim < 0)
		return NULL;

	isl_assert(bmap->ctx, bmap->n_eq == dim, return NULL);
	point = isl_vec_alloc(bmap->ctx, 1 + dim);
	if (!point)
		return NULL;

	isl_int_init(m);

	isl_int_set_si(point->el[0], 1);
	for (j = 0; j < bmap->n_eq; ++j) {
		int i = dim - 1 - j;
		isl_assert(bmap->ctx,
		    !isl_seq_any_non_zero(bmap->eq[j] + 1, i),
		    goto error);
		isl_assert(bmap->ctx,
		    isl_int_is_one(bmap->eq[j][1 + i]) ||
		    isl_int_is_negone(bmap->eq[j][1 + i]),
		    goto error);
		isl_assert(bmap->ctx,
		    !isl_seq_any_non_zero(bmap->eq[j]+1+i+1, dim-i-1),
		    goto error);

````
- **EN**: This block declares or defines routines around `isl_basic_map_dim`, `isl_assert`, `isl_vec_alloc`, `isl_int_init` (+4 more); contains control flow with 1 loop construct(s), 2 conditional check(s); emits return paths that hand results or status codes back to callers; adds defensive checks for invariants or impossible states; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_basic_map_dim`, `isl_assert`, `isl_vec_alloc`, `isl_int_init` (+4 more) 相关的例程; 包含控制流结构：1 处循环、2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 加入用于保护不变量或异常状态的断言/检查；并延续周边实现细节。

### Lines 778-805

````c
		isl_int_gcd(m, point->el[0], bmap->eq[j][1 + i]);
		isl_int_divexact(m, bmap->eq[j][1 + i], m);
		isl_int_abs(m, m);
		isl_seq_scale(point->el, point->el, m, 1 + i);
		isl_int_divexact(m, point->el[0], bmap->eq[j][1 + i]);
		isl_int_neg(m, m);
		isl_int_mul(point->el[1 + i], m, bmap->eq[j][0]);
	}

	isl_int_clear(m);
	return isl_point_alloc(isl_basic_map_get_space(bmap), point);
error:
	isl_int_clear(m);
	isl_vec_free(point);
	return NULL;
}

/* Return isl_bool_true if the singleton map "map1" is a subset of "map2",
 * i.e., if the single element of "map1" is also an element of "map2".
 * Assumes "map2" has known divs.
 */
static isl_bool map_is_singleton_subset(__isl_keep isl_map *map1,
	__isl_keep isl_map *map2)
{
	int i;
	isl_bool is_subset = isl_bool_false;
	struct isl_point *point;

````
- **EN**: This block declares or references types such as `isl_point`; declares or defines routines around `isl_int_gcd`, `isl_int_divexact`, `isl_int_abs`, `isl_seq_scale` (+5 more); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_point`; 声明或定义与 `isl_int_gcd`, `isl_int_divexact`, `isl_int_abs`, `isl_seq_scale` (+5 more) 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 806-826

````c
	if (!map1 || !map2)
		return isl_bool_error;
	if (map1->n != 1)
		isl_die(isl_map_get_ctx(map1), isl_error_invalid,
			"expecting single-disjunct input",
			return isl_bool_error);

	point = singleton_extract_point(map1->p[0]);
	if (!point)
		return isl_bool_error;

	for (i = 0; i < map2->n; ++i) {
		is_subset = isl_basic_map_contains_point(map2->p[i], point);
		if (is_subset)
			break;
	}

	isl_point_free(point);
	return is_subset;
}

````
- **EN**: This block declares or defines routines around `isl_die`, `singleton_extract_point`, `isl_basic_map_contains_point`, `isl_point_free`; contains control flow with 1 loop construct(s), 4 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_die`, `singleton_extract_point`, `isl_basic_map_contains_point`, `isl_point_free` 相关的例程; 包含控制流结构：1 处循环、4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 827-851

````c
static isl_bool map_is_subset(__isl_keep isl_map *map1,
	__isl_keep isl_map *map2)
{
	isl_bool is_subset = isl_bool_false;
	isl_bool empty, single;
	isl_bool rat1, rat2;

	if (!map1 || !map2)
		return isl_bool_error;

	if (!isl_map_has_equal_space(map1, map2))
		return isl_bool_false;

	empty = isl_map_is_empty(map1);
	if (empty < 0)
		return isl_bool_error;
	if (empty)
		return isl_bool_true;

	empty = isl_map_is_empty(map2);
	if (empty < 0)
		return isl_bool_error;
	if (empty)
		return isl_bool_false;

````
- **EN**: This block declares or defines routines around `map_is_subset`, `isl_map_is_empty`; contains control flow with 6 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `map_is_subset`, `isl_map_is_empty` 相关的例程; 包含控制流结构：6 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 852-873

````c
	rat1 = isl_map_has_rational(map1);
	rat2 = isl_map_has_rational(map2);
	if (rat1 < 0 || rat2 < 0)
		return isl_bool_error;
	if (rat1 && !rat2)
		return isl_bool_false;

	if (isl_map_plain_is_universe(map2))
		return isl_bool_true;

	single = isl_map_plain_is_singleton(map1);
	if (single < 0)
		return isl_bool_error;
	map2 = isl_map_compute_divs(isl_map_copy(map2));
	if (single) {
		is_subset = map_is_singleton_subset(map1, map2);
		isl_map_free(map2);
		return is_subset;
	}
	is_subset = map_diff_is_empty(map1, map2);
	isl_map_free(map2);

````
- **EN**: This block declares or defines routines around `isl_map_has_rational`, `isl_map_plain_is_singleton`, `isl_map_compute_divs`, `map_is_singleton_subset` (+2 more); contains control flow with 5 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_map_has_rational`, `isl_map_plain_is_singleton`, `isl_map_compute_divs`, `map_is_singleton_subset` (+2 more) 相关的例程; 包含控制流结构：5 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 874-893

````c
	return is_subset;
}

isl_bool isl_map_is_subset(__isl_keep isl_map *map1, __isl_keep isl_map *map2)
{
	return isl_map_align_params_map_map_and_test(map1, map2,
							&map_is_subset);
}

isl_bool isl_set_is_subset(__isl_keep isl_set *set1, __isl_keep isl_set *set2)
{
	return isl_map_is_subset(set_to_map(set1), set_to_map(set2));
}

__isl_give isl_map *isl_map_make_disjoint(__isl_take isl_map *map)
{
	int i;
	struct isl_subtract_diff_collector sdc;
	sdc.dc.add = &basic_map_subtract_add;

````
- **EN**: This block declares or references types such as `isl_subtract_diff_collector`; declares or defines routines around `isl_map_is_subset`, `isl_set_is_subset`, `isl_map_make_disjoint`; emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或引用类型，例如 `isl_subtract_diff_collector`; 声明或定义与 `isl_map_is_subset`, `isl_set_is_subset`, `isl_map_make_disjoint` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 894-918

````c
	if (!map)
		return NULL;
	if (ISL_F_ISSET(map, ISL_MAP_DISJOINT))
		return map;
	if (map->n <= 1)
		return map;

	map = isl_map_compute_divs(map);
	map = isl_map_remove_empty_parts(map);

	if (!map || map->n <= 1)
		return map;

	sdc.diff = isl_map_from_basic_map(isl_basic_map_copy(map->p[0]));

	for (i = 1; i < map->n; ++i) {
		struct isl_basic_map *bmap = isl_basic_map_copy(map->p[i]);
		struct isl_map *copy = isl_map_copy(sdc.diff);
		if (basic_map_collect_diff(bmap, copy, &sdc.dc) < 0) {
			isl_map_free(sdc.diff);
			sdc.diff = NULL;
			break;
		}
	}

````
- **EN**: This block declares or references types such as `isl_basic_map`, `isl_map`; declares or defines routines around `isl_map_compute_divs`, `isl_map_remove_empty_parts`, `isl_map_from_basic_map`, `isl_basic_map_copy` (+2 more); contains control flow with 1 loop construct(s), 5 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_basic_map`, `isl_map`; 声明或定义与 `isl_map_compute_divs`, `isl_map_remove_empty_parts`, `isl_map_from_basic_map`, `isl_basic_map_copy` (+2 more) 相关的例程; 包含控制流结构：1 处循环、5 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 919-940

````c
	isl_map_free(map);

	return sdc.diff;
}

__isl_give isl_set *isl_set_make_disjoint(__isl_take isl_set *set)
{
	return set_from_map(isl_map_make_disjoint(set_to_map(set)));
}

__isl_give isl_map *isl_map_complement(__isl_take isl_map *map)
{
	isl_map *universe;

	if (!map)
		return NULL;

	universe = isl_map_universe(isl_map_get_space(map));

	return isl_map_subtract(universe, map);
}

````
- **EN**: This block declares or defines routines around `isl_map_free`, `isl_set_make_disjoint`, `isl_map_complement`, `isl_map_universe`; contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_map_free`, `isl_set_make_disjoint`, `isl_map_complement`, `isl_map_universe` 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 941-944

````c
__isl_give isl_set *isl_set_complement(__isl_take isl_set *set)
{
	return isl_map_complement(set);
}
````
- **EN**: This block declares or defines routines around `isl_set_complement`; emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_set_complement` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

## Key Concepts / 关键概念

- **ISL relations and sets**
  - **CN**: ISL 关系与集合
- **Template/type wrappers**
  - **CN**: 模板与类型包装
- **Loop transformation**
  - **CN**: 循环变换
- **Memory access tracking**
  - **CN**: 内存访问跟踪
- **Debug and diagnostics**
  - **CN**: 调试与诊断

## Dependencies / 依赖关系

- **ISL headers**: `isl/set.h`, `isl/map.h` — ISL relation/set primitives that support polyhedral analysis and transformations.
  **ISL headers（CN）**：`isl/set.h`, `isl/map.h` —— 支撑多面体分析与变换的 ISL 关系/集合原语。
- **System/standard headers**: `isl_map_private.h`, `isl_seq.h`, `isl_tab.h`, `isl_point_private.h`, `isl_vec_private.h`, `set_to_map.c`, `set_from_map.c` — Standard-library or system declarations required by the implementation.
  **System/standard headers（CN）**：`isl_map_private.h`, `isl_seq.h`, `isl_tab.h`, `isl_point_private.h`, `isl_vec_private.h`, `set_to_map.c`, `set_from_map.c` —— 实现所需的标准库或系统声明。
