# isl_coalesce.c — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `polly/lib/External/isl/isl_coalesce.c` | `polly/lib/External/isl/isl_coalesce.c` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | Implements or embeds upstream ISL support code used by Polly. | 实现或内嵌 Polly 使用的上游 ISL 支撑代码。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-35

````c
/*
 * Copyright 2008-2009 Katholieke Universiteit Leuven
 * Copyright 2010      INRIA Saclay
 * Copyright 2012-2013 Ecole Normale Superieure
 * Copyright 2014      INRIA Rocquencourt
 * Copyright 2016      INRIA Paris
 * Copyright 2020      Cerebras Systems
 *
 * Use of this software is governed by the MIT license
 *
 * Written by Sven Verdoolaege, K.U.Leuven, Departement
 * Computerwetenschappen, Celestijnenlaan 200A, B-3001 Leuven, Belgium
 * and INRIA Saclay - Ile-de-France, Parc Club Orsay Universite,
 * ZAC des vignes, 4 rue Jacques Monod, 91893 Orsay, France 
 * and Ecole Normale Superieure, 45 rue d’Ulm, 75230 Paris, France
 * and Inria Paris - Rocquencourt, Domaine de Voluceau - Rocquencourt,
 * B.P. 105 - 78153 Le Chesnay, France
 * and Centre de Recherche Inria de Paris, 2 rue Simone Iff - Voie DQ12,
 * CS 42112, 75589 Paris Cedex 12, France
 * and Cerebras Systems, 175 S San Antonio Rd, Los Altos, CA, USA
 */

#include <isl_ctx_private.h>
#include "isl_map_private.h"
#include <isl_seq.h>
#include <isl/options.h>
#include "isl_tab.h"
#include <isl_mat_private.h>
#include <isl_local_space_private.h>
#include <isl_val_private.h>
#include <isl_vec_private.h>
#include <isl_aff_private.h>
#include <isl_equalities.h>
#include <isl_constraint_private.h>

````
- **EN**: This block imports ISL, system/standard headers needed by the surrounding code; uses ISL data structures or helpers for polyhedral reasoning; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 引入周边逻辑所需的 ISL、system/standard 头文件; 使用 ISL 数据结构或辅助函数进行多面体推理; 保留用于解释意图、用法或算法背景的注释.

### Lines 36-73

````c
#include <set_to_map.c>
#include <set_from_map.c>

#define STATUS_ERROR		-1
#define STATUS_REDUNDANT	 1
#define STATUS_VALID	 	 2
#define STATUS_SEPARATE	 	 3
#define STATUS_CUT	 	 4
#define STATUS_ADJ_EQ	 	 5
#define STATUS_ADJ_INEQ	 	 6

static int status_in(isl_int *ineq, struct isl_tab *tab)
{
	enum isl_ineq_type type = isl_tab_ineq_type(tab, ineq);
	switch (type) {
	default:
	case isl_ineq_error:		return STATUS_ERROR;
	case isl_ineq_redundant:	return STATUS_VALID;
	case isl_ineq_separate:		return STATUS_SEPARATE;
	case isl_ineq_cut:		return STATUS_CUT;
	case isl_ineq_adj_eq:		return STATUS_ADJ_EQ;
	case isl_ineq_adj_ineq:		return STATUS_ADJ_INEQ;
	}
}

/* Compute the position of the equalities of basic map "bmap_i"
 * with respect to the basic map represented by "tab_j".
 * The resulting array has twice as many entries as the number
 * of equalities corresponding to the two inequalities to which
 * each equality corresponds.
 */
static int *eq_status_in(__isl_keep isl_basic_map *bmap_i,
	struct isl_tab *tab_j)
{
	int k, l;
	int *eq;
	isl_size dim;

````
- **EN**: This block imports system/standard headers needed by the surrounding code; uses preprocessor directives to control compilation, macros, or include guards; declares or references types such as `isl_tab`; defines enum values such as `isl_ineq_type`; and continues the surrounding implementation details.
- **CN**: 该代码块 引入周边逻辑所需的 system/standard 头文件; 使用预处理指令控制编译、宏或头文件保护; 声明或引用类型，例如 `isl_tab`; 定义枚举类型，例如 `isl_ineq_type`；并延续周边实现细节。

### Lines 74-107

````c
	dim = isl_basic_map_dim(bmap_i, isl_dim_all);
	if (dim < 0)
		return NULL;

	eq = isl_calloc_array(bmap_i->ctx, int, 2 * bmap_i->n_eq);
	if (!eq)
		return NULL;

	for (k = 0; k < bmap_i->n_eq; ++k) {
		for (l = 0; l < 2; ++l) {
			isl_seq_neg(bmap_i->eq[k], bmap_i->eq[k], 1+dim);
			eq[2 * k + l] = status_in(bmap_i->eq[k], tab_j);
			if (eq[2 * k + l] == STATUS_ERROR)
				goto error;
		}
	}

	return eq;
error:
	free(eq);
	return NULL;
}

/* Compute the position of the inequalities of basic map "bmap_i"
 * (also represented by "tab_i", if not NULL) with respect to the basic map
 * represented by "tab_j".
 */
static int *ineq_status_in(__isl_keep isl_basic_map *bmap_i,
	struct isl_tab *tab_i, struct isl_tab *tab_j)
{
	int k;
	unsigned n_eq = bmap_i->n_eq;
	int *ineq = isl_calloc_array(bmap_i->ctx, int, bmap_i->n_ineq);

````
- **EN**: This block declares or references types such as `isl_tab`; declares or defines routines around `isl_basic_map_dim`, `isl_calloc_array`, `isl_seq_neg`, `status_in` (+2 more); contains control flow with 2 loop construct(s), 3 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_tab`; 声明或定义与 `isl_basic_map_dim`, `isl_calloc_array`, `isl_seq_neg`, `status_in` (+2 more) 相关的例程; 包含控制流结构：2 处循环、3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 108-138

````c
	if (!ineq)
		return NULL;

	for (k = 0; k < bmap_i->n_ineq; ++k) {
		if (tab_i && isl_tab_is_redundant(tab_i, n_eq + k)) {
			ineq[k] = STATUS_REDUNDANT;
			continue;
		}
		ineq[k] = status_in(bmap_i->ineq[k], tab_j);
		if (ineq[k] == STATUS_ERROR)
			goto error;
		if (ineq[k] == STATUS_SEPARATE)
			break;
	}

	return ineq;
error:
	free(ineq);
	return NULL;
}

static int any(int *con, unsigned len, int status)
{
	int i;

	for (i = 0; i < len ; ++i)
		if (con[i] == status)
			return 1;
	return 0;
}

````
- **EN**: This block declares or defines routines around `status_in`, `free`, `any`; contains control flow with 2 loop construct(s), 5 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `status_in`, `free`, `any` 相关的例程; 包含控制流结构：2 处循环、5 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 139-166

````c
/* Return the first position of "status" in the list "con" of length "len".
 * Return -1 if there is no such entry.
 */
static int find(int *con, unsigned len, int status)
{
	int i;

	for (i = 0; i < len ; ++i)
		if (con[i] == status)
			return i;
	return -1;
}

static int count(int *con, unsigned len, int status)
{
	int i;
	int c = 0;

	for (i = 0; i < len ; ++i)
		if (con[i] == status)
			c++;
	return c;
}

static int all(int *con, unsigned len, int status)
{
	int i;

````
- **EN**: This block declares or defines routines around `find`, `count`, `all`; contains control flow with 2 loop construct(s), 2 conditional check(s); emits return paths that hand results or status codes back to callers; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `find`, `count`, `all` 相关的例程; 包含控制流结构：2 处循环、2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 保留用于解释意图、用法或算法背景的注释.

### Lines 167-198

````c
	for (i = 0; i < len ; ++i) {
		if (con[i] == STATUS_REDUNDANT)
			continue;
		if (con[i] != status)
			return 0;
	}
	return 1;
}

/* Internal information associated to a basic map in a map
 * that is to be coalesced by isl_map_coalesce.
 *
 * "bmap" is the basic map itself (or NULL if "removed" is set)
 * "tab" is the corresponding tableau (or NULL if "removed" is set)
 * "hull_hash" identifies the affine space in which "bmap" lives.
 * "modified" is set if this basic map may not be identical
 * to any of the basic maps in the input.
 * "removed" is set if this basic map has been removed from the map
 * "simplify" is set if this basic map may have some unknown integer
 * divisions that were not present in the input basic maps.  The basic
 * map should then be simplified such that we may be able to find
 * a definition among the constraints.
 *
 * "eq" and "ineq" are only set if we are currently trying to coalesce
 * this basic map with another basic map, in which case they represent
 * the position of the inequalities of this basic map with respect to
 * the other basic map.  The number of elements in the "eq" array
 * is twice the number of equalities in the "bmap", corresponding
 * to the two inequalities that make up each equality.
 */
struct isl_coalesce_info {
	isl_basic_map *bmap;
````
- **EN**: This block declares or references types such as `isl_coalesce_info`; declares or defines routines around `itself`, `tableau`; contains control flow with 1 loop construct(s), 2 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_coalesce_info`; 声明或定义与 `itself`, `tableau` 相关的例程; 包含控制流结构：1 处循环、2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 199-227

````c
	struct isl_tab *tab;
	uint32_t hull_hash;
	int modified;
	int removed;
	int simplify;
	int *eq;
	int *ineq;
};

/* Is there any (half of an) equality constraint in the description
 * of the basic map represented by "info" that
 * has position "status" with respect to the other basic map?
 */
static int any_eq(struct isl_coalesce_info *info, int status)
{
	isl_size n_eq;

	n_eq = isl_basic_map_n_equality(info->bmap);
	return any(info->eq, 2 * n_eq, status);
}

/* Is there any inequality constraint in the description
 * of the basic map represented by "info" that
 * has position "status" with respect to the other basic map?
 */
static int any_ineq(struct isl_coalesce_info *info, int status)
{
	isl_size n_ineq;

````
- **EN**: This block declares or references types such as `isl_tab`; declares or defines routines around `any`, `any_eq`, `isl_basic_map_n_equality`, `any_ineq`; emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_tab`; 声明或定义与 `any`, `any_eq`, `isl_basic_map_n_equality`, `any_ineq` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 228-255

````c
	n_ineq = isl_basic_map_n_inequality(info->bmap);
	return any(info->ineq, n_ineq, status);
}

/* Return the position of the first half on an equality constraint
 * in the description of the basic map represented by "info" that
 * has position "status" with respect to the other basic map.
 * The returned value is twice the position of the equality constraint
 * plus zero for the negative half and plus one for the positive half.
 * Return -1 if there is no such entry.
 */
static int find_eq(struct isl_coalesce_info *info, int status)
{
	isl_size n_eq;

	n_eq = isl_basic_map_n_equality(info->bmap);
	return find(info->eq, 2 * n_eq, status);
}

/* Return the position of the first inequality constraint in the description
 * of the basic map represented by "info" that
 * has position "status" with respect to the other basic map.
 * Return -1 if there is no such entry.
 */
static int find_ineq(struct isl_coalesce_info *info, int status)
{
	isl_size n_ineq;

````
- **EN**: This block declares or defines routines around `isl_basic_map_n_inequality`, `find_eq`, `isl_basic_map_n_equality`, `find_ineq`; emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `isl_basic_map_n_inequality`, `find_eq`, `isl_basic_map_n_equality`, `find_ineq` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理; 保留用于解释意图、用法或算法背景的注释.

### Lines 256-283

````c
	n_ineq = isl_basic_map_n_inequality(info->bmap);
	return find(info->ineq, n_ineq, status);
}

/* Return the number of (halves of) equality constraints in the description
 * of the basic map represented by "info" that
 * have position "status" with respect to the other basic map.
 */
static int count_eq(struct isl_coalesce_info *info, int status)
{
	isl_size n_eq;

	n_eq = isl_basic_map_n_equality(info->bmap);
	return count(info->eq, 2 * n_eq, status);
}

/* Return the number of inequality constraints in the description
 * of the basic map represented by "info" that
 * have position "status" with respect to the other basic map.
 */
static int count_ineq(struct isl_coalesce_info *info, int status)
{
	isl_size n_ineq;

	n_ineq = isl_basic_map_n_inequality(info->bmap);
	return count(info->ineq, n_ineq, status);
}

````
- **EN**: This block declares or defines routines around `isl_basic_map_n_inequality`, `of`, `count_eq`, `isl_basic_map_n_equality` (+1 more); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `isl_basic_map_n_inequality`, `of`, `count_eq`, `isl_basic_map_n_equality` (+1 more) 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理; 保留用于解释意图、用法或算法背景的注释.

### Lines 284-310

````c
/* Are all non-redundant constraints of the basic map represented by "info"
 * either valid or cut constraints with respect to the other basic map?
 */
static int all_valid_or_cut(struct isl_coalesce_info *info)
{
	int i;

	for (i = 0; i < 2 * info->bmap->n_eq; ++i) {
		if (info->eq[i] == STATUS_REDUNDANT)
			continue;
		if (info->eq[i] == STATUS_VALID)
			continue;
		if (info->eq[i] == STATUS_CUT)
			continue;
		return 0;
	}

	for (i = 0; i < info->bmap->n_ineq; ++i) {
		if (info->ineq[i] == STATUS_REDUNDANT)
			continue;
		if (info->ineq[i] == STATUS_VALID)
			continue;
		if (info->ineq[i] == STATUS_CUT)
			continue;
		return 0;
	}

````
- **EN**: This block declares or defines routines around `all_valid_or_cut`; contains control flow with 2 loop construct(s), 6 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `all_valid_or_cut` 相关的例程; 包含控制流结构：2 处循环、6 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 311-342

````c
	return 1;
}

/* Compute the hash of the (apparent) affine hull of info->bmap (with
 * the existentially quantified variables removed) and store it
 * in info->hash.
 */
static int coalesce_info_set_hull_hash(struct isl_coalesce_info *info)
{
	isl_basic_map *hull;
	isl_size n_div;

	hull = isl_basic_map_copy(info->bmap);
	hull = isl_basic_map_plain_affine_hull(hull);
	n_div = isl_basic_map_dim(hull, isl_dim_div);
	if (n_div < 0)
		hull = isl_basic_map_free(hull);
	hull = isl_basic_map_drop_constraints_involving_dims(hull,
							isl_dim_div, 0, n_div);
	info->hull_hash = isl_basic_map_get_hash(hull);
	isl_basic_map_free(hull);

	return hull ? 0 : -1;
}

/* Free all the allocated memory in an array
 * of "n" isl_coalesce_info elements.
 */
static void clear_coalesce_info(int n, struct isl_coalesce_info *info)
{
	int i;

````
- **EN**: This block declares or defines routines around `the`, `coalesce_info_set_hull_hash`, `isl_basic_map_copy`, `isl_basic_map_plain_affine_hull` (+5 more); contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `the`, `coalesce_info_set_hull_hash`, `isl_basic_map_copy`, `isl_basic_map_plain_affine_hull` (+5 more) 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 343-372

````c
	if (!info)
		return;

	for (i = 0; i < n; ++i) {
		isl_basic_map_free(info[i].bmap);
		isl_tab_free(info[i].tab);
	}

	free(info);
}

/* Clear the memory associated to "info".
 */
static void clear(struct isl_coalesce_info *info)
{
	info->bmap = isl_basic_map_free(info->bmap);
	isl_tab_free(info->tab);
	info->tab = NULL;
}

/* Drop the basic map represented by "info".
 * That is, clear the memory associated to the entry and
 * mark it as having been removed.
 */
static void drop(struct isl_coalesce_info *info)
{
	clear(info);
	info->removed = 1;
}

````
- **EN**: This block declares or defines routines around `isl_basic_map_free`, `isl_tab_free`, `free`, `clear` (+1 more); contains control flow with 1 loop construct(s), 1 conditional check(s); uses ISL data structures or helpers for polyhedral reasoning; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `isl_basic_map_free`, `isl_tab_free`, `free`, `clear` (+1 more) 相关的例程; 包含控制流结构：1 处循环、1 处条件判断; 使用 ISL 数据结构或辅助函数进行多面体推理; 保留用于解释意图、用法或算法背景的注释.

### Lines 373-400

````c
/* Exchange the information in "info1" with that in "info2".
 */
static void exchange(struct isl_coalesce_info *info1,
	struct isl_coalesce_info *info2)
{
	struct isl_coalesce_info info;

	info = *info1;
	*info1 = *info2;
	*info2 = info;
}

/* This type represents the kind of change that has been performed
 * while trying to coalesce two basic maps.
 *
 * isl_change_none: nothing was changed
 * isl_change_drop_first: the first basic map was removed
 * isl_change_drop_second: the second basic map was removed
 * isl_change_fuse: the two basic maps were replaced by a new basic map.
 */
enum isl_change {
	isl_change_error = -1,
	isl_change_none = 0,
	isl_change_drop_first,
	isl_change_drop_second,
	isl_change_fuse,
};

````
- **EN**: This block declares or references types such as `isl_coalesce_info`; defines enum values such as `isl_change`; declares or defines routines around `exchange`; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_coalesce_info`; 定义枚举类型，例如 `isl_change`; 声明或定义与 `exchange` 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 401-433

````c
/* Update "change" based on an interchange of the first and the second
 * basic map.  That is, interchange isl_change_drop_first and
 * isl_change_drop_second.
 */
static enum isl_change invert_change(enum isl_change change)
{
	switch (change) {
	case isl_change_error:
		return isl_change_error;
	case isl_change_none:
		return isl_change_none;
	case isl_change_drop_first:
		return isl_change_drop_second;
	case isl_change_drop_second:
		return isl_change_drop_first;
	case isl_change_fuse:
		return isl_change_fuse;
	}

	return isl_change_error;
}

/* Add the valid constraints of the basic map represented by "info"
 * to "bmap".  "len" is the size of the constraints.
 * If only one of the pair of inequalities that make up an equality
 * is valid, then add that inequality.
 */
static __isl_give isl_basic_map *add_valid_constraints(
	__isl_take isl_basic_map *bmap, struct isl_coalesce_info *info,
	unsigned len)
{
	int k, l;

````
- **EN**: This block declares or defines routines around `invert_change`, `add_valid_constraints`; contains control flow with 1 switch dispatch(es); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `invert_change`, `add_valid_constraints` 相关的例程; 包含控制流结构：1 处分支派发; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 434-465

````c
	if (!bmap)
		return NULL;

	for (k = 0; k < info->bmap->n_eq; ++k) {
		if (info->eq[2 * k] == STATUS_VALID &&
		    info->eq[2 * k + 1] == STATUS_VALID) {
			l = isl_basic_map_alloc_equality(bmap);
			if (l < 0)
				return isl_basic_map_free(bmap);
			isl_seq_cpy(bmap->eq[l], info->bmap->eq[k], len);
		} else if (info->eq[2 * k] == STATUS_VALID) {
			l = isl_basic_map_alloc_inequality(bmap);
			if (l < 0)
				return isl_basic_map_free(bmap);
			isl_seq_neg(bmap->ineq[l], info->bmap->eq[k], len);
		} else if (info->eq[2 * k + 1] == STATUS_VALID) {
			l = isl_basic_map_alloc_inequality(bmap);
			if (l < 0)
				return isl_basic_map_free(bmap);
			isl_seq_cpy(bmap->ineq[l], info->bmap->eq[k], len);
		}
	}

	for (k = 0; k < info->bmap->n_ineq; ++k) {
		if (info->ineq[k] != STATUS_VALID)
			continue;
		l = isl_basic_map_alloc_inequality(bmap);
		if (l < 0)
			return isl_basic_map_free(bmap);
		isl_seq_cpy(bmap->ineq[l], info->bmap->ineq[k], len);
	}

````
- **EN**: This block declares or defines routines around `isl_basic_map_alloc_equality`, `isl_seq_cpy`, `isl_basic_map_alloc_inequality`, `isl_seq_neg`; contains control flow with 2 loop construct(s), 9 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_basic_map_alloc_equality`, `isl_seq_cpy`, `isl_basic_map_alloc_inequality`, `isl_seq_neg` 相关的例程; 包含控制流结构：2 处循环、9 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 466-497

````c
	return bmap;
}

/* Is "bmap" defined by a number of (non-redundant) constraints that
 * is greater than the number of constraints of basic maps i and j combined?
 * Equalities are counted as two inequalities.
 */
static int number_of_constraints_increases(int i, int j,
	struct isl_coalesce_info *info,
	__isl_keep isl_basic_map *bmap, struct isl_tab *tab)
{
	int k, n_old, n_new;

	n_old = 2 * info[i].bmap->n_eq + info[i].bmap->n_ineq;
	n_old += 2 * info[j].bmap->n_eq + info[j].bmap->n_ineq;

	n_new = 2 * bmap->n_eq;
	for (k = 0; k < bmap->n_ineq; ++k)
		if (!isl_tab_is_redundant(tab, bmap->n_eq + k))
			++n_new;

	return n_new > n_old;
}

/* Replace the pair of basic maps i and j by the basic map bounded
 * by the valid constraints in both basic maps and the constraints
 * in extra (if not NULL).
 * Place the fused basic map in the position that is the smallest of i and j.
 *
 * If "detect_equalities" is set, then look for equalities encoded
 * as pairs of inequalities.
 * If "check_number" is set, then the original basic maps are only
````
- **EN**: This block declares or references types such as `isl_coalesce_info`; declares or defines routines around `of`, `number_of_constraints_increases`, `extra`; contains control flow with 1 loop construct(s), 1 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_coalesce_info`; 声明或定义与 `of`, `number_of_constraints_increases`, `extra` 相关的例程; 包含控制流结构：1 处循环、1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 498-526

````c
 * replaced if the total number of constraints does not increase.
 * While the number of integer divisions in the two basic maps
 * is assumed to be the same, the actual definitions may be different.
 * We only copy the definition from one of the basic maps if it is
 * the same as that of the other basic map.  Otherwise, we mark
 * the integer division as unknown and simplify the basic map
 * in an attempt to recover the integer division definition.
 * If any extra constraints get introduced, then these may
 * involve integer divisions with a unit coefficient.
 * Eliminate those that do not appear with any other coefficient
 * in other constraints, to ensure they get eliminated completely,
 * improving the chances of further coalescing.
 *
 * Factor out any (hidden) common factor from the constraint
 * coefficients of the fused basic map
 * to improve the detection of adjacent constraints
 * with respect to other basic maps.
 */
static enum isl_change fuse(int i, int j, struct isl_coalesce_info *info,
	__isl_keep isl_mat *extra, int detect_equalities, int check_number)
{
	int k, l;
	struct isl_basic_map *fused = NULL;
	struct isl_tab *fused_tab = NULL;
	isl_size total = isl_basic_map_dim(info[i].bmap, isl_dim_all);
	unsigned extra_rows = extra ? extra->n_row : 0;
	unsigned n_eq, n_ineq;
	int simplify = 0;

````
- **EN**: This block declares or references types such as `isl_basic_map`, `isl_tab`; declares or defines routines around `any`, `fuse`, `isl_basic_map_dim`; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或引用类型，例如 `isl_basic_map`, `isl_tab`; 声明或定义与 `any`, `fuse`, `isl_basic_map_dim` 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 527-557

````c
	if (total < 0)
		return isl_change_error;
	if (j < i)
		return fuse(j, i, info, extra, detect_equalities, check_number);

	n_eq = info[i].bmap->n_eq + info[j].bmap->n_eq;
	n_ineq = info[i].bmap->n_ineq + info[j].bmap->n_ineq;
	fused = isl_basic_map_alloc_space(isl_space_copy(info[i].bmap->dim),
		    info[i].bmap->n_div, n_eq, n_eq + n_ineq + extra_rows);
	fused = add_valid_constraints(fused, &info[i], 1 + total);
	fused = add_valid_constraints(fused, &info[j], 1 + total);
	if (!fused)
		goto error;
	if (ISL_F_ISSET(info[i].bmap, ISL_BASIC_MAP_RATIONAL) &&
	    ISL_F_ISSET(info[j].bmap, ISL_BASIC_MAP_RATIONAL))
		ISL_F_SET(fused, ISL_BASIC_MAP_RATIONAL);

	for (k = 0; k < info[i].bmap->n_div; ++k) {
		int l = isl_basic_map_alloc_div(fused);
		if (l < 0)
			goto error;
		if (isl_seq_eq(info[i].bmap->div[k], info[j].bmap->div[k],
				1 + 1 + total)) {
			isl_seq_cpy(fused->div[l], info[i].bmap->div[k],
				1 + 1 + total);
		} else {
			isl_int_set_si(fused->div[l][0], 0);
			simplify = 1;
		}
	}

````
- **EN**: This block declares or defines routines around `isl_basic_map_alloc_space`, `add_valid_constraints`, `ISL_F_ISSET`, `ISL_F_SET` (+3 more); contains control flow with 1 loop construct(s), 6 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_basic_map_alloc_space`, `add_valid_constraints`, `ISL_F_ISSET`, `ISL_F_SET` (+3 more) 相关的例程; 包含控制流结构：1 处循环、6 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 558-587

````c
	for (k = 0; k < extra_rows; ++k) {
		l = isl_basic_map_alloc_inequality(fused);
		if (l < 0)
			goto error;
		isl_seq_cpy(fused->ineq[l], extra->row[k], 1 + total);
	}

	if (detect_equalities)
		fused = isl_basic_map_detect_inequality_pairs(fused, NULL);
	fused = isl_basic_map_gauss(fused, NULL);
	if (simplify || info[j].simplify) {
		fused = isl_basic_map_simplify(fused);
		info[i].simplify = 0;
	} else if (extra_rows > 0) {
		fused = isl_basic_map_eliminate_pure_unit_divs(fused);
	}
	fused = isl_basic_map_finalize(fused);
	fused = isl_basic_map_reduce_coefficients(fused);

	fused_tab = isl_tab_from_basic_map(fused, 0);
	if (isl_tab_detect_redundant(fused_tab) < 0)
		goto error;

	if (check_number &&
	    number_of_constraints_increases(i, j, info, fused, fused_tab)) {
		isl_tab_free(fused_tab);
		isl_basic_map_free(fused);
		return isl_change_none;
	}

````
- **EN**: This block declares or defines routines around `isl_basic_map_alloc_inequality`, `isl_seq_cpy`, `isl_basic_map_detect_inequality_pairs`, `isl_basic_map_gauss` (+8 more); contains control flow with 1 loop construct(s), 6 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_basic_map_alloc_inequality`, `isl_seq_cpy`, `isl_basic_map_detect_inequality_pairs`, `isl_basic_map_gauss` (+8 more) 相关的例程; 包含控制流结构：1 处循环、6 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 588-619

````c
	clear(&info[i]);
	info[i].bmap = fused;
	info[i].tab = fused_tab;
	info[i].modified = 1;
	drop(&info[j]);

	return isl_change_fuse;
error:
	isl_tab_free(fused_tab);
	isl_basic_map_free(fused);
	return isl_change_error;
}

/* Given a pair of basic maps i and j such that all constraints are either
 * "valid" or "cut", check if the facets corresponding to the "cut"
 * constraints of i lie entirely within basic map j.
 * If so, replace the pair by the basic map consisting of the valid
 * constraints in both basic maps.
 * Checking whether the facet lies entirely within basic map j
 * is performed by checking whether the constraints of basic map j
 * are valid for the facet.  These tests are performed on a rational
 * tableau to avoid the theoretical possibility that a constraint
 * that was considered to be a cut constraint for the entire basic map i
 * happens to be considered to be a valid constraint for the facet,
 * even though it cuts off the same rational points.
 *
 * To see that we are not introducing any extra points, call the
 * two basic maps A and B and the resulting map U and let x
 * be an element of U \setminus ( A \cup B ).
 * A line connecting x with an element of A \cup B meets a facet F
 * of either A or B.  Assume it is a facet of B and let c_1 be
 * the corresponding facet constraint.  We have c_1(x) < 0 and
````
- **EN**: This block declares or defines routines around `clear`, `drop`, `isl_tab_free`, `isl_basic_map_free` (+2 more); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `clear`, `drop`, `isl_tab_free`, `isl_basic_map_free` (+2 more) 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理; 保留用于解释意图、用法或算法背景的注释.

### Lines 620-651

````c
 * so c_1 is a cut constraint.  This implies that there is some
 * (possibly rational) point x' satisfying the constraints of A
 * and the opposite of c_1 as otherwise c_1 would have been marked
 * valid for A.  The line connecting x and x' meets a facet of A
 * in a (possibly rational) point that also violates c_1, but this
 * is impossible since all cut constraints of B are valid for all
 * cut facets of A.
 * In case F is a facet of A rather than B, then we can apply the
 * above reasoning to find a facet of B separating x from A \cup B first.
 */
static enum isl_change check_facets(int i, int j,
	struct isl_coalesce_info *info)
{
	int k, l;
	struct isl_tab_undo *snap, *snap2;
	unsigned n_eq = info[i].bmap->n_eq;

	snap = isl_tab_snap(info[i].tab);
	if (isl_tab_mark_rational(info[i].tab) < 0)
		return isl_change_error;
	snap2 = isl_tab_snap(info[i].tab);

	for (k = 0; k < info[i].bmap->n_ineq; ++k) {
		if (info[i].ineq[k] != STATUS_CUT)
			continue;
		if (isl_tab_select_facet(info[i].tab, n_eq + k) < 0)
			return isl_change_error;
		for (l = 0; l < info[j].bmap->n_ineq; ++l) {
			int stat;
			if (info[j].ineq[l] != STATUS_CUT)
				continue;
			stat = status_in(info[j].bmap->ineq[l], info[i].tab);
````
- **EN**: This block declares or references types such as `isl_coalesce_info`, `isl_tab_undo`; declares or defines routines around `a`, `check_facets`, `isl_tab_snap`, `status_in`; contains control flow with 2 loop construct(s), 4 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_coalesce_info`, `isl_tab_undo`; 声明或定义与 `a`, `check_facets`, `isl_tab_snap`, `status_in` 相关的例程; 包含控制流结构：2 处循环、4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 652-682

````c
			if (stat < 0)
				return isl_change_error;
			if (stat != STATUS_VALID)
				break;
		}
		if (isl_tab_rollback(info[i].tab, snap2) < 0)
			return isl_change_error;
		if (l < info[j].bmap->n_ineq)
			break;
	}

	if (k < info[i].bmap->n_ineq) {
		if (isl_tab_rollback(info[i].tab, snap) < 0)
			return isl_change_error;
		return isl_change_none;
	}
	return fuse(i, j, info, NULL, 0, 0);
}

/* Check if info->bmap contains the basic map represented
 * by the tableau "tab".
 * For each equality, we check both the constraint itself
 * (as an inequality) and its negation.  Make sure the
 * equality is returned to its original state before returning.
 */
static isl_bool contains(struct isl_coalesce_info *info, struct isl_tab *tab)
{
	int k;
	isl_size dim;
	isl_basic_map *bmap = info->bmap;

````
- **EN**: This block declares or defines routines around `contains`; contains control flow with 6 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `contains` 相关的例程; 包含控制流结构：6 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 683-714

````c
	dim = isl_basic_map_dim(bmap, isl_dim_all);
	if (dim < 0)
		return isl_bool_error;
	for (k = 0; k < bmap->n_eq; ++k) {
		int stat;
		isl_seq_neg(bmap->eq[k], bmap->eq[k], 1 + dim);
		stat = status_in(bmap->eq[k], tab);
		isl_seq_neg(bmap->eq[k], bmap->eq[k], 1 + dim);
		if (stat < 0)
			return isl_bool_error;
		if (stat != STATUS_VALID)
			return isl_bool_false;
		stat = status_in(bmap->eq[k], tab);
		if (stat < 0)
			return isl_bool_error;
		if (stat != STATUS_VALID)
			return isl_bool_false;
	}

	for (k = 0; k < bmap->n_ineq; ++k) {
		int stat;
		if (info->ineq[k] == STATUS_REDUNDANT)
			continue;
		stat = status_in(bmap->ineq[k], tab);
		if (stat < 0)
			return isl_bool_error;
		if (stat != STATUS_VALID)
			return isl_bool_false;
	}
	return isl_bool_true;
}

````
- **EN**: This block declares or defines routines around `isl_basic_map_dim`, `isl_seq_neg`, `status_in`; contains control flow with 2 loop construct(s), 8 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_basic_map_dim`, `isl_seq_neg`, `status_in` 相关的例程; 包含控制流结构：2 处循环、8 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 715-746

````c
/* Basic map "i" has an inequality "k" that is adjacent
 * to some inequality of basic map "j".  All the other inequalities
 * are valid for "j".
 * If not NULL, then "extra" contains extra wrapping constraints that are valid
 * for both "i" and "j".
 * Check if basic map "j" forms an extension of basic map "i",
 * taking into account the extra constraints, if any.
 *
 * Note that this function is only called if some of the equalities or
 * inequalities of basic map "j" do cut basic map "i".  The function is
 * correct even if there are no such cut constraints, but in that case
 * the additional checks performed by this function are overkill.
 *
 * In particular, we replace constraint k, say f >= 0, by constraint
 * f <= -1, add the inequalities of "j" that are valid for "i",
 * as well as the "extra" constraints, if any,
 * and check if the result is a subset of basic map "j".
 * To improve the chances of the subset relation being detected,
 * any variable that only attains a single integer value
 * in the tableau of "i" is first fixed to that value.
 * If the result is a subset, then we know that this result is exactly equal
 * to basic map "j" since all its constraints are valid for basic map "j".
 * By combining the valid constraints of "i" (all equalities and all
 * inequalities except "k"), the valid constraints of "j" and
 * the "extra" constraints, if any, we therefore
 * obtain a basic map that is equal to their union.
 * In this case, there is no need to perform a rollback of the tableau
 * since it is going to be destroyed in fuse().
 *
 *
 *	|\__			|\__
 *	|   \__			|   \__
````
- **EN**: This block declares or defines routines around `fuse`; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `fuse` 相关的例程; 保留用于解释意图、用法或算法背景的注释.

### Lines 747-774

````c
 *	|      \_	=>	|      \__
 *	|_______| _		|_________\
 *
 *
 *	|\			|\
 *	| \			| \
 *	|  \			|  \
 *	|  |			|   \
 *	|  ||\		=>      |    \
 *	|  || \			|     \
 *	|  ||  |		|      |
 *	|__||_/			|_____/
 *
 *
 *	_______			 _______
 *     |       | __		|       \__
 *     |       ||__|	=>	|        __|
 *     |_______|		|_______/
 */
static enum isl_change is_adj_ineq_extension_with_wraps(int i, int j, int k,
	struct isl_coalesce_info *info, __isl_keep isl_mat *extra)
{
	struct isl_tab_undo *snap;
	isl_size n_eq_i, n_ineq_j, n_extra;
	isl_size total = isl_basic_map_dim(info[i].bmap, isl_dim_all);
	isl_stat r;
	isl_bool super;

````
- **EN**: This block declares or references types such as `isl_coalesce_info`, `isl_tab_undo`; declares or defines routines around `is_adj_ineq_extension_with_wraps`, `isl_basic_map_dim`; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或引用类型，例如 `isl_coalesce_info`, `isl_tab_undo`; 声明或定义与 `is_adj_ineq_extension_with_wraps`, `isl_basic_map_dim` 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 775-812

````c
	if (total < 0)
		return isl_change_error;

	n_eq_i = isl_basic_map_n_equality(info[i].bmap);
	n_ineq_j = isl_basic_map_n_inequality(info[j].bmap);
	n_extra = isl_mat_rows(extra);
	if (n_eq_i < 0 || n_ineq_j < 0 || n_extra < 0)
		return isl_change_error;

	if (isl_tab_extend_cons(info[i].tab, 1 + n_ineq_j + n_extra) < 0)
		return isl_change_error;

	snap = isl_tab_snap(info[i].tab);

	if (isl_tab_unrestrict(info[i].tab, n_eq_i + k) < 0)
		return isl_change_error;

	isl_seq_neg(info[i].bmap->ineq[k], info[i].bmap->ineq[k], 1 + total);
	isl_int_sub_ui(info[i].bmap->ineq[k][0], info[i].bmap->ineq[k][0], 1);
	r = isl_tab_add_ineq(info[i].tab, info[i].bmap->ineq[k]);
	isl_seq_neg(info[i].bmap->ineq[k], info[i].bmap->ineq[k], 1 + total);
	isl_int_sub_ui(info[i].bmap->ineq[k][0], info[i].bmap->ineq[k][0], 1);
	if (r < 0)
		return isl_change_error;

	for (k = 0; k < n_ineq_j; ++k) {
		if (info[j].ineq[k] != STATUS_VALID)
			continue;
		if (isl_tab_add_ineq(info[i].tab, info[j].bmap->ineq[k]) < 0)
			return isl_change_error;
	}
	for (k = 0; k < n_extra; ++k) {
		if (isl_tab_add_ineq(info[i].tab, extra->row[k]) < 0)
			return isl_change_error;
	}
	if (isl_tab_detect_constants(info[i].tab) < 0)
		return isl_change_error;

````
- **EN**: This block declares or defines routines around `isl_basic_map_n_equality`, `isl_basic_map_n_inequality`, `isl_mat_rows`, `isl_tab_snap` (+3 more); contains control flow with 2 loop construct(s), 9 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_basic_map_n_equality`, `isl_basic_map_n_inequality`, `isl_mat_rows`, `isl_tab_snap` (+3 more) 相关的例程; 包含控制流结构：2 处循环、9 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 813-844

````c
	super = contains(&info[j], info[i].tab);
	if (super < 0)
		return isl_change_error;
	if (super)
		return fuse(i, j, info, extra, 0, 0);

	if (isl_tab_rollback(info[i].tab, snap) < 0)
		return isl_change_error;

	return isl_change_none;
}

/* Given an affine transformation matrix "T", does row "row" represent
 * anything other than a unit vector (possibly shifted by a constant)
 * that is not involved in any of the other rows?
 *
 * That is, if a constraint involves the variable corresponding to
 * the row, then could its preimage by "T" have any coefficients
 * that are different from those in the original constraint?
 */
static int not_unique_unit_row(__isl_keep isl_mat *T, int row)
{
	int i, j;
	int len = T->n_col - 1;

	i = isl_seq_first_non_zero(T->row[row] + 1, len);
	if (i < 0)
		return 1;
	if (!isl_int_is_one(T->row[row][1 + i]) &&
	    !isl_int_is_negone(T->row[row][1 + i]))
		return 1;

````
- **EN**: This block declares or defines routines around `contains`, `vector`, `not_unique_unit_row`, `isl_seq_first_non_zero` (+1 more); contains control flow with 5 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `contains`, `vector`, `not_unique_unit_row`, `isl_seq_first_non_zero` (+1 more) 相关的例程; 包含控制流结构：5 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 845-875

````c
	j = isl_seq_first_non_zero(T->row[row] + 1 + i + 1, len - (i + 1));
	if (j >= 0)
		return 1;

	for (j = 1; j < T->n_row; ++j) {
		if (j == row)
			continue;
		if (!isl_int_is_zero(T->row[j][1 + i]))
			return 1;
	}

	return 0;
}

/* Does inequality constraint "ineq" of "bmap" involve any of
 * the variables marked in "affected"?
 * "total" is the total number of variables, i.e., the number
 * of entries in "affected".
 */
static isl_bool is_affected(__isl_keep isl_basic_map *bmap, int ineq,
	int *affected, int total)
{
	int i;

	for (i = 0; i < total; ++i) {
		if (!affected[i])
			continue;
		if (!isl_int_is_zero(bmap->ineq[ineq][1 + i]))
			return isl_bool_true;
	}

````
- **EN**: This block declares or defines routines around `isl_seq_first_non_zero`, `is_affected`; contains control flow with 2 loop construct(s), 5 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_seq_first_non_zero`, `is_affected` 相关的例程; 包含控制流结构：2 处循环、5 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 876-904

````c
	return isl_bool_false;
}

/* Given the compressed version of inequality constraint "ineq"
 * of info->bmap in "v", check if the constraint can be tightened,
 * where the compression is based on an equality constraint valid
 * for info->tab.
 * If so, add the tightened version of the inequality constraint
 * to info->tab.  "v" may be modified by this function.
 *
 * That is, if the compressed constraint is of the form
 *
 *	m f() + c >= 0
 *
 * with 0 < c < m, then it is equivalent to
 *
 *	f() >= 0
 *
 * This means that c can also be subtracted from the original,
 * uncompressed constraint without affecting the integer points
 * in info->tab.  Add this tightened constraint as an extra row
 * to info->tab to make this information explicitly available.
 */
static __isl_give isl_vec *try_tightening(struct isl_coalesce_info *info,
	int ineq, __isl_take isl_vec *v)
{
	isl_ctx *ctx;
	isl_stat r;

````
- **EN**: This block declares or defines routines around `f`, `try_tightening`; emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `f`, `try_tightening` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理; 保留用于解释意图、用法或算法背景的注释.

### Lines 905-931

````c
	if (!v)
		return NULL;

	ctx = isl_vec_get_ctx(v);
	isl_seq_gcd(v->el + 1, v->size - 1, &ctx->normalize_gcd);
	if (isl_int_is_zero(ctx->normalize_gcd) ||
	    isl_int_is_one(ctx->normalize_gcd)) {
		return v;
	}

	v = isl_vec_cow(v);
	if (!v)
		return NULL;

	isl_int_fdiv_r(v->el[0], v->el[0], ctx->normalize_gcd);
	if (isl_int_is_zero(v->el[0]))
		return v;

	if (isl_tab_extend_cons(info->tab, 1) < 0)
		return isl_vec_free(v);

	isl_int_sub(info->bmap->ineq[ineq][0],
		    info->bmap->ineq[ineq][0], v->el[0]);
	r = isl_tab_add_ineq(info->tab, info->bmap->ineq[ineq]);
	isl_int_add(info->bmap->ineq[ineq][0],
		    info->bmap->ineq[ineq][0], v->el[0]);

````
- **EN**: This block declares or defines routines around `isl_vec_get_ctx`, `isl_seq_gcd`, `isl_int_is_one`, `isl_vec_cow` (+4 more); contains control flow with 5 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_vec_get_ctx`, `isl_seq_gcd`, `isl_int_is_one`, `isl_vec_cow` (+4 more) 相关的例程; 包含控制流结构：5 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 932-963

````c
	if (r < 0)
		return isl_vec_free(v);

	return v;
}

/* Tighten the (non-redundant) constraints on the facet represented
 * by info->tab.
 * In particular, on input, info->tab represents the result
 * of relaxing the "n" inequality constraints of info->bmap in "relaxed"
 * by one, i.e., replacing f_i >= 0 by f_i + 1 >= 0, and then
 * replacing the one at index "l" by the corresponding equality,
 * i.e., f_k + 1 = 0, with k = relaxed[l].
 *
 * Compute a variable compression from the equality constraint f_k + 1 = 0
 * and use it to tighten the other constraints of info->bmap
 * (that is, all constraints that have not been relaxed),
 * updating info->tab (and leaving info->bmap untouched).
 * The compression handles essentially two cases, one where a variable
 * is assigned a fixed value and can therefore be eliminated, and one
 * where one variable is a shifted multiple of some other variable and
 * can therefore be replaced by that multiple.
 * Gaussian elimination would also work for the first case, but for
 * the second case, the effectiveness would depend on the order
 * of the variables.
 * After compression, some of the constraints may have coefficients
 * with a common divisor.  If this divisor does not divide the constant
 * term, then the constraint can be tightened.
 * The tightening is performed on the tableau info->tab by introducing
 * extra (temporary) constraints.
 *
 * Only constraints that are possibly affected by the compression are
````
- **EN**: This block declares or defines routines around `the`, `tab`, `extra`; contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `the`, `tab`, `extra` 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 964-1001

````c
 * considered.  In particular, if the constraint only involves variables
 * that are directly mapped to a distinct set of other variables, then
 * no common divisor can be introduced and no tightening can occur.
 *
 * It is important to only consider the non-redundant constraints
 * since the facet constraint has been relaxed prior to the call
 * to this function, meaning that the constraints that were redundant
 * prior to the relaxation may no longer be redundant.
 * These constraints will be ignored in the fused result, so
 * the fusion detection should not exploit them.
 */
static isl_stat tighten_on_relaxed_facet(struct isl_coalesce_info *info,
	int n, int *relaxed, int l)
{
	isl_size total;
	isl_ctx *ctx;
	isl_vec *v = NULL;
	isl_mat *T;
	int i;
	int k;
	int *affected;

	k = relaxed[l];
	ctx = isl_basic_map_get_ctx(info->bmap);
	total = isl_basic_map_dim(info->bmap, isl_dim_all);
	if (total < 0)
		return isl_stat_error;
	isl_int_add_ui(info->bmap->ineq[k][0], info->bmap->ineq[k][0], 1);
	T = isl_mat_sub_alloc6(ctx, info->bmap->ineq, k, 1, 0, 1 + total);
	T = isl_mat_variable_compression(T, NULL);
	isl_int_sub_ui(info->bmap->ineq[k][0], info->bmap->ineq[k][0], 1);
	if (!T)
		return isl_stat_error;
	if (T->n_col == 0) {
		isl_mat_free(T);
		return isl_stat_ok;
	}

````
- **EN**: This block declares or defines routines around `tighten_on_relaxed_facet`, `isl_basic_map_get_ctx`, `isl_basic_map_dim`, `isl_int_add_ui` (+4 more); contains control flow with 3 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `tighten_on_relaxed_facet`, `isl_basic_map_get_ctx`, `isl_basic_map_dim`, `isl_int_add_ui` (+4 more) 相关的例程; 包含控制流结构：3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1002-1030

````c
	affected = isl_alloc_array(ctx, int, total);
	if (!affected)
		goto error;

	for (i = 0; i < total; ++i)
		affected[i] = not_unique_unit_row(T, 1 + i);

	for (i = 0; i < info->bmap->n_ineq; ++i) {
		isl_bool handle;
		if (any(relaxed, n, i))
			continue;
		if (info->ineq[i] == STATUS_REDUNDANT)
			continue;
		handle = is_affected(info->bmap, i, affected, total);
		if (handle < 0)
			goto error;
		if (!handle)
			continue;
		v = isl_vec_alloc(ctx, 1 + total);
		if (!v)
			goto error;
		isl_seq_cpy(v->el, info->bmap->ineq[i], 1 + total);
		v = isl_vec_mat_product(v, isl_mat_copy(T));
		v = try_tightening(info, i, v);
		isl_vec_free(v);
		if (!v)
			goto error;
	}

````
- **EN**: This block declares or defines routines around `isl_alloc_array`, `not_unique_unit_row`, `is_affected`, `isl_vec_alloc` (+4 more); contains control flow with 2 loop construct(s), 7 conditional check(s); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_alloc_array`, `not_unique_unit_row`, `is_affected`, `isl_vec_alloc` (+4 more) 相关的例程; 包含控制流结构：2 处循环、7 处条件判断; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1031-1056

````c
	isl_mat_free(T);
	free(affected);
	return isl_stat_ok;
error:
	isl_mat_free(T);
	free(affected);
	return isl_stat_error;
}

/* Replace the basic maps "i" and "j" by an extension of "i"
 * along the "n" inequality constraints in "relax" by one.
 * The tableau info[i].tab has already been extended.
 * Extend info[i].bmap accordingly by relaxing all constraints in "relax"
 * by one.
 * Each integer division that does not have exactly the same
 * definition in "i" and "j" is marked unknown and the basic map
 * is scheduled to be simplified in an attempt to recover
 * the integer division definition.
 * Place the extension in the position that is the smallest of i and j.
 */
static enum isl_change extend(int i, int j, int n, int *relax,
	struct isl_coalesce_info *info)
{
	int l;
	isl_size total;

````
- **EN**: This block declares or references types such as `isl_coalesce_info`; declares or defines routines around `isl_mat_free`, `free`, `extend`; emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_coalesce_info`; 声明或定义与 `isl_mat_free`, `free`, `extend` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 1057-1088

````c
	info[i].bmap = isl_basic_map_cow(info[i].bmap);
	total = isl_basic_map_dim(info[i].bmap, isl_dim_all);
	if (total < 0)
		return isl_change_error;
	for (l = 0; l < info[i].bmap->n_div; ++l)
		if (!isl_seq_eq(info[i].bmap->div[l],
				info[j].bmap->div[l], 1 + 1 + total)) {
			isl_int_set_si(info[i].bmap->div[l][0], 0);
			info[i].simplify = 1;
		}
	for (l = 0; l < n; ++l)
		isl_int_add_ui(info[i].bmap->ineq[relax[l]][0],
				info[i].bmap->ineq[relax[l]][0], 1);
	ISL_F_CLR(info[i].bmap, ISL_BASIC_MAP_NO_REDUNDANT);
	ISL_F_SET(info[i].bmap, ISL_BASIC_MAP_FINAL);
	drop(&info[j]);
	info[i].modified = 1;
	if (j < i)
		exchange(&info[i], &info[j]);
	return isl_change_fuse;
}

/* Basic map "i" has "n" inequality constraints (collected in "relax")
 * that are such that they include basic map "j" if they are relaxed
 * by one.  All the other inequalities are valid for "j".
 * Check if basic map "j" forms an extension of basic map "i".
 *
 * In particular, relax the constraints in "relax", compute the corresponding
 * facets one by one and check whether each of these is included
 * in the other basic map.
 * Before testing for inclusion, the constraints on each facet
 * are tightened to increase the chance of an inclusion being detected.
````
- **EN**: This block declares or defines routines around `isl_basic_map_cow`, `isl_basic_map_dim`, `isl_int_set_si`, `isl_int_add_ui` (+5 more); contains control flow with 2 loop construct(s), 3 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_basic_map_cow`, `isl_basic_map_dim`, `isl_int_set_si`, `isl_int_add_ui` (+5 more) 相关的例程; 包含控制流结构：2 处循环、3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 1089-1120

````c
 * (Adding the valid constraints of "j" to the tableau of "i", as is done
 * in is_adj_ineq_extension, may further increase those chances, but this
 * is not currently done.)
 * If each facet is included, we know that relaxing the constraints extends
 * the basic map with exactly the other basic map (we already know that this
 * other basic map is included in the extension, because all other
 * inequality constraints are valid of "j") and we can replace the
 * two basic maps by this extension.
 *
 * If any of the relaxed constraints turn out to be redundant, then bail out.
 * isl_tab_select_facet refuses to handle such constraints.  It may be
 * possible to handle them anyway by making a distinction between
 * redundant constraints with a corresponding facet that still intersects
 * the set (allowing isl_tab_select_facet to handle them) and
 * those where the facet does not intersect the set (which can be ignored
 * because the empty facet is trivially included in the other disjunct).
 * However, relaxed constraints that turn out to be redundant should
 * be fairly rare and no such instance has been reported where
 * coalescing would be successful.
 *        ____			  _____
 *       /    || 		 /     |
 *      /     ||  		/      |
 *      \     ||   	=>	\      |
 *       \    ||		 \     |
 *        \___||		  \____|
 *
 *
 *	 \			|\
 *	|\\			| \
 *	| \\			|  \
 *	|  |		=>	|  /
 *	| /			| /
````
- **EN**: This block declares or defines routines around `map`, `set`; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `map`, `set` 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1121-1152

````c
 *	|/			|/
 */
static enum isl_change is_relaxed_extension(int i, int j, int n, int *relax,
	struct isl_coalesce_info *info)
{
	int l;
	isl_bool super;
	struct isl_tab_undo *snap, *snap2;
	unsigned n_eq = info[i].bmap->n_eq;

	for (l = 0; l < n; ++l)
		if (isl_tab_is_equality(info[i].tab, n_eq + relax[l]))
			return isl_change_none;

	snap = isl_tab_snap(info[i].tab);
	for (l = 0; l < n; ++l)
		if (isl_tab_relax(info[i].tab, n_eq + relax[l]) < 0)
			return isl_change_error;
	for (l = 0; l < n; ++l) {
		if (!isl_tab_is_redundant(info[i].tab, n_eq + relax[l]))
			continue;
		if (isl_tab_rollback(info[i].tab, snap) < 0)
			return isl_change_error;
		return isl_change_none;
	}
	snap2 = isl_tab_snap(info[i].tab);
	for (l = 0; l < n; ++l) {
		if (isl_tab_rollback(info[i].tab, snap2) < 0)
			return isl_change_error;
		if (isl_tab_select_facet(info[i].tab, n_eq + relax[l]) < 0)
			return isl_change_error;
		if (tighten_on_relaxed_facet(&info[i], n, relax, l) < 0)
````
- **EN**: This block declares or references types such as `isl_coalesce_info`, `isl_tab_undo`; declares or defines routines around `is_relaxed_extension`, `isl_tab_snap`; contains control flow with 4 loop construct(s), 7 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_coalesce_info`, `isl_tab_undo`; 声明或定义与 `is_relaxed_extension`, `isl_tab_snap` 相关的例程; 包含控制流结构：4 处循环、7 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 1153-1183

````c
			return isl_change_error;
		super = contains(&info[j], info[i].tab);
		if (super < 0)
			return isl_change_error;
		if (super)
			continue;
		if (isl_tab_rollback(info[i].tab, snap) < 0)
			return isl_change_error;
		return isl_change_none;
	}

	if (isl_tab_rollback(info[i].tab, snap2) < 0)
		return isl_change_error;
	return extend(i, j, n, relax, info);
}

/* Data structure that keeps track of the wrapping constraints
 * and of information to bound the coefficients of those constraints.
 *
 * "failed" is set if wrapping has failed.
 * bound is set if we want to apply a bound on the coefficients
 * mat contains the wrapping constraints
 * max is the bound on the coefficients (if bound is set)
 */
struct isl_wraps {
	int failed;
	int bound;
	isl_mat *mat;
	isl_int max;
};

````
- **EN**: This block declares or references types such as `isl_wraps`; declares or defines routines around `contains`, `coefficients`; contains control flow with 4 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_wraps`; 声明或定义与 `contains`, `coefficients` 相关的例程; 包含控制流结构：4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 1184-1216

````c
/* Update wraps->max to be greater than or equal to the coefficients
 * in the equalities and inequalities of info->bmap that can be removed
 * if we end up applying wrapping.
 */
static isl_stat wraps_update_max(struct isl_wraps *wraps,
	struct isl_coalesce_info *info)
{
	int k;
	isl_int max_k;
	isl_size total = isl_basic_map_dim(info->bmap, isl_dim_all);

	if (total < 0)
		return isl_stat_error;
	isl_int_init(max_k);

	for (k = 0; k < info->bmap->n_eq; ++k) {
		if (info->eq[2 * k] == STATUS_VALID &&
		    info->eq[2 * k + 1] == STATUS_VALID)
			continue;
		isl_seq_abs_max(info->bmap->eq[k] + 1, total, &max_k);
		if (isl_int_abs_gt(max_k, wraps->max))
			isl_int_set(wraps->max, max_k);
	}

	for (k = 0; k < info->bmap->n_ineq; ++k) {
		if (info->ineq[k] == STATUS_VALID ||
		    info->ineq[k] == STATUS_REDUNDANT)
			continue;
		isl_seq_abs_max(info->bmap->ineq[k] + 1, total, &max_k);
		if (isl_int_abs_gt(max_k, wraps->max))
			isl_int_set(wraps->max, max_k);
	}

````
- **EN**: This block declares or references types such as `isl_coalesce_info`; declares or defines routines around `wraps_update_max`, `isl_basic_map_dim`, `isl_int_init`, `isl_seq_abs_max` (+1 more); contains control flow with 2 loop construct(s), 5 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_coalesce_info`; 声明或定义与 `wraps_update_max`, `isl_basic_map_dim`, `isl_int_init`, `isl_seq_abs_max` (+1 more) 相关的例程; 包含控制流结构：2 处循环、5 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 1217-1249

````c
	isl_int_clear(max_k);

	return isl_stat_ok;
}

/* Initialize the isl_wraps data structure.
 * If we want to bound the coefficients of the wrapping constraints,
 * we set wraps->max to the largest coefficient
 * in the equalities and inequalities that can be removed if we end up
 * applying wrapping.
 */
static isl_stat wraps_init(struct isl_wraps *wraps, __isl_take isl_mat *mat,
	struct isl_coalesce_info *info, int i, int j)
{
	isl_ctx *ctx;

	wraps->failed = 0;
	wraps->bound = 0;
	wraps->mat = mat;
	if (!mat)
		return isl_stat_error;
	wraps->mat->n_row = 0;
	ctx = isl_mat_get_ctx(mat);
	wraps->bound = isl_options_get_coalesce_bounded_wrapping(ctx);
	if (!wraps->bound)
		return isl_stat_ok;
	isl_int_init(wraps->max);
	isl_int_set_si(wraps->max, 0);
	if (wraps_update_max(wraps, &info[i]) < 0)
		return isl_stat_error;
	if (wraps_update_max(wraps, &info[j]) < 0)
		return isl_stat_error;

````
- **EN**: This block declares or references types such as `isl_coalesce_info`; declares or defines routines around `isl_int_clear`, `wraps_init`, `isl_mat_get_ctx`, `isl_options_get_coalesce_bounded_wrapping` (+2 more); contains control flow with 4 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_coalesce_info`; 声明或定义与 `isl_int_clear`, `wraps_init`, `isl_mat_get_ctx`, `isl_options_get_coalesce_bounded_wrapping` (+2 more) 相关的例程; 包含控制流结构：4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 1250-1278

````c
	return isl_stat_ok;
}

/* Free the contents of the isl_wraps data structure.
 */
static void wraps_free(struct isl_wraps *wraps)
{
	isl_mat_free(wraps->mat);
	if (wraps->bound)
		isl_int_clear(wraps->max);
}

/* Mark the wrapping as failed.
 */
static isl_stat wraps_mark_failed(struct isl_wraps *wraps)
{
	wraps->failed = 1;
	return isl_stat_ok;
}

/* Is the wrapping constraint in row "row" allowed?
 *
 * If wraps->bound is set, we check that none of the coefficients
 * is greater than wraps->max.
 */
static int allow_wrap(struct isl_wraps *wraps, int row)
{
	int i;

````
- **EN**: This block declares or defines routines around `wraps_free`, `isl_mat_free`, `isl_int_clear`, `wraps_mark_failed` (+1 more); contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `wraps_free`, `isl_mat_free`, `isl_int_clear`, `wraps_mark_failed` (+1 more) 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 1279-1314

````c
	if (!wraps->bound)
		return 1;

	for (i = 1; i < wraps->mat->n_col; ++i)
		if (isl_int_abs_gt(wraps->mat->row[row][i], wraps->max))
			return 0;

	return 1;
}

/* Wrap "ineq" (or its opposite if "negate" is set) around "bound"
 * to include "set" and add the result in position "w" of "wraps".
 * "len" is the total number of coefficients in "bound" and "ineq".
 * Return isl_bool_true on success, isl_bool_false on failure and
 * isl_bool_error on error.
 * Wrapping can fail if the result of wrapping is equal to "bound"
 * or if we want to bound the sizes of the coefficients and
 * the wrapped constraint does not satisfy this bound.
 */
static isl_bool add_wrap(struct isl_wraps *wraps, int w, isl_int *bound,
	isl_int *ineq, unsigned len, __isl_keep isl_set *set, int negate)
{
	isl_seq_cpy(wraps->mat->row[w], bound, len);
	if (negate) {
		isl_seq_neg(wraps->mat->row[w + 1], ineq, len);
		ineq = wraps->mat->row[w + 1];
	}
	if (!isl_set_wrap_facet(set, wraps->mat->row[w], ineq))
		return isl_bool_error;
	if (isl_seq_eq(wraps->mat->row[w], bound, len))
		return isl_bool_false;
	if (!allow_wrap(wraps, w))
		return isl_bool_false;
	return isl_bool_true;
}

````
- **EN**: This block declares or defines routines around `add_wrap`, `isl_seq_cpy`, `isl_seq_neg`; contains control flow with 1 loop construct(s), 6 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `add_wrap`, `isl_seq_cpy`, `isl_seq_neg` 相关的例程; 包含控制流结构：1 处循环、6 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 1315-1346

````c
/* This function has two modes of operations.
 *
 * If "add_valid" is set, then all the constraints of info->bmap
 * (except the opposite of "bound") are valid for the other basic map.
 * In this case, attempts are made to wrap some of these valid constraints
 * to more tightly fit around "set".  Only successful wrappings are recorded
 * and failed wrappings are ignored.
 *
 * If "add_valid" is not set, then some of the constraints of info->bmap
 * are not valid for the other basic map, and only those are considered
 * for wrapping.  In this case all attempted wrappings need to succeed.
 * Otherwise "wraps" is marked as failed.
 * Note that the constraints that are valid for the other basic map
 * will be added to the combined basic map by default, so there is
 * no need to wrap them.
 * The caller wrap_in_facets even relies on this function not wrapping
 * any constraints that are already valid.
 *
 * Only consider constraints that are not redundant (as determined
 * by info->tab) and that are valid or invalid depending on "add_valid".
 * Wrap each constraint around "bound" such that it includes the whole
 * set "set" and append the resulting constraint to "wraps".
 * "wraps" is assumed to have been pre-allocated to the appropriate size.
 * wraps->n_row is the number of actual wrapped constraints that have
 * been added.
 * If any of the wrapping problems results in a constraint that is
 * identical to "bound", then this means that "set" is unbounded in such
 * a way that no wrapping is possible.
 * Similarly, if we want to bound the coefficients of the wrapping
 * constraints and a newly added wrapping constraint does not
 * satisfy the bound, then the wrapping is considered to have failed.
 * Note though that "wraps" is only marked failed if "add_valid" is not set.
````
- **EN**: This block declares or defines routines around `redundant`; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `redundant` 相关的例程; 保留用于解释意图、用法或算法背景的注释.

### Lines 1347-1375

````c
 */
static isl_stat add_selected_wraps(struct isl_wraps *wraps,
	struct isl_coalesce_info *info, isl_int *bound, __isl_keep isl_set *set,
	int add_valid)
{
	int l, m;
	int w;
	isl_bool added;
	isl_basic_map *bmap = info->bmap;
	isl_size total = isl_basic_map_dim(bmap, isl_dim_all);
	unsigned len = 1 + total;

	if (total < 0)
		return isl_stat_error;

	w = wraps->mat->n_row;

	for (l = 0; l < bmap->n_ineq; ++l) {
		int is_valid = info->ineq[l] == STATUS_VALID;
		if ((!add_valid && is_valid) ||
		    info->ineq[l] == STATUS_REDUNDANT)
			continue;
		if (isl_seq_is_neg(bound, bmap->ineq[l], len))
			continue;
		if (isl_seq_eq(bound, bmap->ineq[l], len))
			continue;
		if (isl_tab_is_redundant(info->tab, bmap->n_eq + l))
			continue;

````
- **EN**: This block declares or references types such as `isl_coalesce_info`; declares or defines routines around `add_selected_wraps`, `isl_basic_map_dim`; contains control flow with 1 loop construct(s), 5 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_coalesce_info`; 声明或定义与 `add_selected_wraps`, `isl_basic_map_dim` 相关的例程; 包含控制流结构：1 处循环、5 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 1376-1402

````c
		added = add_wrap(wraps, w, bound, bmap->ineq[l], len, set, 0);
		if (added < 0)
			return isl_stat_error;
		if (!added && !is_valid)
			goto unbounded;
		if (added)
			++w;
	}
	for (l = 0; l < bmap->n_eq; ++l) {
		if (isl_seq_is_neg(bound, bmap->eq[l], len))
			continue;
		if (isl_seq_eq(bound, bmap->eq[l], len))
			continue;

		for (m = 0; m < 2; ++m) {
			if (info->eq[2 * l + m] == STATUS_VALID)
				continue;
			added = add_wrap(wraps, w, bound, bmap->eq[l], len,
					set, !m);
			if (added < 0)
				return isl_stat_error;
			if (!added)
				goto unbounded;
			++w;
		}
	}

````
- **EN**: This block declares or defines routines around `add_wrap`; contains control flow with 2 loop construct(s), 8 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `add_wrap` 相关的例程; 包含控制流结构：2 处循环、8 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1403-1434

````c
	wraps->mat->n_row = w;
	return isl_stat_ok;
unbounded:
	return wraps_mark_failed(wraps);
}

/* For each constraint in info->bmap that is not redundant (as determined
 * by info->tab) and that is not a valid constraint for the other basic map,
 * wrap the constraint around "bound" such that it includes the whole
 * set "set" and append the resulting constraint to "wraps".
 * Note that the constraints that are valid for the other basic map
 * will be added to the combined basic map by default, so there is
 * no need to wrap them.
 * The caller wrap_in_facets even relies on this function not wrapping
 * any constraints that are already valid.
 * "wraps" is assumed to have been pre-allocated to the appropriate size.
 * wraps->n_row is the number of actual wrapped constraints that have
 * been added.
 * If any of the wrapping problems results in a constraint that is
 * identical to "bound", then this means that "set" is unbounded in such
 * a way that no wrapping is possible.  If this happens then "wraps"
 * is marked as failed.
 * Similarly, if we want to bound the coefficients of the wrapping
 * constraints and a newly added wrapping constraint does not
 * satisfy the bound, then "wraps" is also marked as failed.
 */
static isl_stat add_wraps(struct isl_wraps *wraps,
	struct isl_coalesce_info *info, isl_int *bound, __isl_keep isl_set *set)
{
	return add_selected_wraps(wraps, info, bound, set, 0);
}

````
- **EN**: This block declares or references types such as `isl_coalesce_info`; declares or defines routines around `redundant`, `add_wraps`; emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_coalesce_info`; 声明或定义与 `redundant`, `add_wraps` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 1435-1462

````c
/* Check if the constraints in "wraps" from "first" until the last
 * are all valid for the basic set represented by "tab",
 * dropping the invalid constraints if "keep" is set and
 * marking the wrapping as failed if "keep" is not set and
 * any constraint turns out to be invalid.
 */
static isl_stat check_wraps(struct isl_wraps *wraps, int first,
	struct isl_tab *tab, int keep)
{
	int i;

	for (i = wraps->mat->n_row - 1; i >= first; --i) {
		enum isl_ineq_type type;
		type = isl_tab_ineq_type(tab, wraps->mat->row[i]);
		if (type == isl_ineq_error)
			return isl_stat_error;
		if (type == isl_ineq_redundant)
			continue;
		if (!keep)
			return wraps_mark_failed(wraps);
		wraps->mat = isl_mat_drop_rows(wraps->mat, i, 1);
		if (!wraps->mat)
			return isl_stat_error;
	}

	return isl_stat_ok;
}

````
- **EN**: This block declares or references types such as `isl_tab`; defines enum values such as `isl_ineq_type`; declares or defines routines around `check_wraps`, `isl_tab_ineq_type`, `isl_mat_drop_rows`; contains control flow with 1 loop construct(s), 4 conditional check(s); and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_tab`; 定义枚举类型，例如 `isl_ineq_type`; 声明或定义与 `check_wraps`, `isl_tab_ineq_type`, `isl_mat_drop_rows` 相关的例程; 包含控制流结构：1 处循环、4 处条件判断；并延续周边实现细节。

### Lines 1463-1490

````c
/* Return a set that corresponds to the non-redundant constraints
 * (as recorded in info->tab) of info->bmap.
 *
 * It's important to remove the redundant constraints as some
 * of the other constraints may have been modified after the
 * constraints were marked redundant.
 * In particular, a constraint may have been relaxed.
 * Redundant constraints are ignored when a constraint is relaxed
 * and should therefore continue to be ignored ever after.
 * Otherwise, the relaxation might be thwarted by some of
 * these constraints.
 *
 * Update the underlying set to ensure that the dimension doesn't change.
 * Otherwise the integer divisions could get dropped if the tab
 * turns out to be empty.
 */
static __isl_give isl_set *set_from_updated_bmap(struct isl_coalesce_info *info)
{
	isl_basic_map *bmap;
	isl_basic_set *bset;

	bmap = isl_basic_map_copy(info->bmap);
	bset = isl_basic_map_underlying_set(bmap);
	bset = isl_basic_set_cow(bset);
	bset = isl_basic_set_update_from_tab(bset, info->tab);
	return isl_set_from_basic_set(bset);
}

````
- **EN**: This block declares or defines routines around `set_from_updated_bmap`, `isl_basic_map_copy`, `isl_basic_map_underlying_set`, `isl_basic_set_cow` (+1 more); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `set_from_updated_bmap`, `isl_basic_map_copy`, `isl_basic_map_underlying_set`, `isl_basic_set_cow` (+1 more) 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理; 保留用于解释意图、用法或算法背景的注释.

### Lines 1491-1516

````c
/* Does "info" have any cut constraints that are redundant?
 */
static isl_bool has_redundant_cuts(struct isl_coalesce_info *info)
{
	int l;
	isl_size n_eq, n_ineq;

	n_eq = isl_basic_map_n_equality(info->bmap);
	n_ineq = isl_basic_map_n_inequality(info->bmap);
	if (n_eq < 0 || n_ineq < 0)
		return isl_bool_error;
	for (l = 0; l < n_ineq; ++l) {
		int red;

		if (info->ineq[l] != STATUS_CUT)
			continue;
		red = isl_tab_is_redundant(info->tab, n_eq + l);
		if (red < 0)
			return isl_bool_error;
		if (red)
			return isl_bool_true;
	}

	return isl_bool_false;
}

````
- **EN**: This block declares or defines routines around `has_redundant_cuts`, `isl_basic_map_n_equality`, `isl_basic_map_n_inequality`, `isl_tab_is_redundant`; contains control flow with 1 loop construct(s), 4 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `has_redundant_cuts`, `isl_basic_map_n_equality`, `isl_basic_map_n_inequality`, `isl_tab_is_redundant` 相关的例程; 包含控制流结构：1 处循环、4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 1517-1553

````c
/* Wrap some constraints of info->bmap that bound the facet defined
 * by inequality "k" around (the opposite of) this inequality to
 * include "set".  "bound" may be used to store the negated inequality.
 *
 * If "add_valid" is set, then all ridges are already valid and
 * the purpose is to wrap "set" more tightly.  In this case,
 * wrapping doesn't fail, although it is possible that no constraint
 * gets wrapped.
 *
 * If "add_valid" is not set, then some of the ridges are cut constraints
 * and only those are wrapped around "set".
 *
 * Since the wrapped constraints are not guaranteed to contain the whole
 * of info->bmap, we check them in check_wraps.
 * If any of the wrapped constraints turn out to be invalid, then
 * check_wraps will mark "wraps" as failed if "add_valid" is not set.
 * If "add_valid" is set, then the offending constraints are
 * simply removed.
 *
 * If the facet turns out to be empty, then no wrapping can be performed.
 * This is considered a failure, unless "add_valid" is set.
 *
 * If any of the cut constraints of info->bmap turn out
 * to be redundant with respect to other constraints
 * then these will neither be wrapped nor added directly to the result.
 * The result may therefore not be correct.
 * Skip wrapping and mark "wraps" as failed in this case.
 */
static isl_stat add_selected_wraps_around_facet(struct isl_wraps *wraps,
	struct isl_coalesce_info *info, int k, isl_int *bound,
	__isl_keep isl_set *set, int add_valid)
{
	isl_bool nowrap;
	struct isl_tab_undo *snap;
	int n;
	isl_size total = isl_basic_map_dim(info->bmap, isl_dim_all);

````
- **EN**: This block declares or references types such as `isl_coalesce_info`, `isl_tab_undo`; declares or defines routines around `around`, `add_selected_wraps_around_facet`, `isl_basic_map_dim`; uses ISL data structures or helpers for polyhedral reasoning; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或引用类型，例如 `isl_coalesce_info`, `isl_tab_undo`; 声明或定义与 `around`, `add_selected_wraps_around_facet`, `isl_basic_map_dim` 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理; 保留用于解释意图、用法或算法背景的注释.

### Lines 1554-1581

````c
	if (total < 0)
		return isl_stat_error;

	snap = isl_tab_snap(info->tab);

	if (isl_tab_select_facet(info->tab, info->bmap->n_eq + k) < 0)
		return isl_stat_error;
	if (isl_tab_detect_redundant(info->tab) < 0)
		return isl_stat_error;
	if (info->tab->empty) {
		if (isl_tab_rollback(info->tab, snap) < 0)
			return isl_stat_error;
		if (!add_valid)
			return wraps_mark_failed(wraps);
		return isl_stat_ok;
	}
	nowrap = has_redundant_cuts(info);
	if (nowrap < 0)
		return isl_stat_error;

	n = wraps->mat->n_row;
	if (!nowrap) {
		isl_seq_neg(bound, info->bmap->ineq[k], 1 + total);

		if (add_selected_wraps(wraps, info, bound, set, add_valid) < 0)
			return isl_stat_error;
	}

````
- **EN**: This block declares or defines routines around `isl_tab_snap`, `has_redundant_cuts`, `isl_seq_neg`; contains control flow with 9 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_tab_snap`, `has_redundant_cuts`, `isl_seq_neg` 相关的例程; 包含控制流结构：9 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1582-1618

````c
	if (isl_tab_rollback(info->tab, snap) < 0)
		return isl_stat_error;
	if (nowrap)
		return wraps_mark_failed(wraps);
	if (check_wraps(wraps, n, info->tab, add_valid) < 0)
		return isl_stat_error;

	return isl_stat_ok;
}

/* Wrap the constraints of info->bmap that bound the facet defined
 * by inequality "k" around (the opposite of) this inequality to
 * include "set".  "bound" may be used to store the negated inequality.
 * If any of the wrapped constraints turn out to be invalid for info->bmap
 * itself, then mark "wraps" as failed.
 */
static isl_stat add_wraps_around_facet(struct isl_wraps *wraps,
	struct isl_coalesce_info *info, int k, isl_int *bound,
	__isl_keep isl_set *set)
{
	return add_selected_wraps_around_facet(wraps, info, k, bound, set, 0);
}

/* Wrap the (valid) constraints of info->bmap that bound the facet defined
 * by inequality "k" around (the opposite of) this inequality to
 * include "set" more tightly.
 * "bound" may be used to store the negated inequality.
 * Remove any wrapping constraints that turn out to be invalid
 * for info->bmap itself.
 */
static isl_stat add_valid_wraps_around_facet(struct isl_wraps *wraps,
	struct isl_coalesce_info *info, int k, isl_int *bound,
	__isl_keep isl_set *set)
{
	return add_selected_wraps_around_facet(wraps, info, k, bound, set, 1);
}

````
- **EN**: This block declares or references types such as `isl_coalesce_info`; declares or defines routines around `around`, `add_wraps_around_facet`, `the`, `add_valid_wraps_around_facet`; contains control flow with 3 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_coalesce_info`; 声明或定义与 `around`, `add_wraps_around_facet`, `the`, `add_valid_wraps_around_facet` 相关的例程; 包含控制流结构：3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 1619-1647

````c
/* Basic map "i" has an inequality (say "k") that is adjacent
 * to some inequality of basic map "j".  All the other inequalities
 * are valid for "j".
 * Check if basic map "j" forms an extension of basic map "i".
 *
 * Note that this function is only called if some of the equalities or
 * inequalities of basic map "j" do cut basic map "i".  The function is
 * correct even if there are no such cut constraints, but in that case
 * the additional checks performed by this function are overkill.
 *
 * First try and wrap the ridges of "k" around "j".
 * Note that those ridges are already valid for "j",
 * but the wrapped versions may wrap "j" more tightly,
 * increasing the chances of "j" being detected as an extension of "i"
 */
static enum isl_change is_adj_ineq_extension(int i, int j,
	struct isl_coalesce_info *info)
{
	int k;
	enum isl_change change;
	isl_size total;
	isl_size n_eq_i, n_ineq_i;
	struct isl_wraps wraps;
	isl_ctx *ctx;
	isl_mat *mat;
	isl_vec *bound;
	isl_set *set_j;
	isl_stat r;

````
- **EN**: This block declares or references types such as `isl_coalesce_info`, `isl_wraps`; defines enum values such as `isl_change`; declares or defines routines around `inequality`, `is_adj_ineq_extension`; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_coalesce_info`, `isl_wraps`; 定义枚举类型，例如 `isl_change`; 声明或定义与 `inequality`, `is_adj_ineq_extension` 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 1648-1673

````c
	k = find_ineq(&info[i], STATUS_ADJ_INEQ);
	if (k < 0)
		isl_die(isl_basic_map_get_ctx(info[i].bmap), isl_error_internal,
			"info[i].ineq should have exactly one STATUS_ADJ_INEQ",
			return isl_change_error);

	total = isl_basic_map_dim(info[i].bmap, isl_dim_all);
	n_eq_i = isl_basic_map_n_equality(info[i].bmap);
	n_ineq_i = isl_basic_map_n_inequality(info[i].bmap);
	if (total < 0 || n_eq_i < 0 || n_ineq_i < 0)
		return isl_change_error;

	set_j = set_from_updated_bmap(&info[j]);
	ctx = isl_basic_map_get_ctx(info[i].bmap);
	bound = isl_vec_alloc(ctx, 1 + total);
	mat = isl_mat_alloc(ctx, 2 * n_eq_i + n_ineq_i, 1 + total);
	if (wraps_init(&wraps, mat, info, i, j) < 0)
		goto error;
	if (!bound || !set_j)
		goto error;
	r = add_valid_wraps_around_facet(&wraps, &info[i], k, bound->el, set_j);
	if (r < 0)
		goto error;

	change = is_adj_ineq_extension_with_wraps(i, j, k, info, wraps.mat);

````
- **EN**: This block declares or defines routines around `find_ineq`, `isl_die`, `isl_basic_map_dim`, `isl_basic_map_n_equality` (+7 more); contains control flow with 5 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `find_ineq`, `isl_die`, `isl_basic_map_dim`, `isl_basic_map_n_equality` (+7 more) 相关的例程; 包含控制流结构：5 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1674-1705

````c
	wraps_free(&wraps);
	isl_vec_free(bound);
	isl_set_free(set_j);

	return change;
error:
	wraps_free(&wraps);
	isl_vec_free(bound);
	isl_set_free(set_j);
	return isl_change_error;
}

/* Both basic maps have at least one inequality with and adjacent
 * (but opposite) inequality in the other basic map.
 * Check that there are no cut constraints and that there is only
 * a single pair of adjacent inequalities.
 * If so, we can replace the pair by a single basic map described
 * by all but the pair of adjacent inequalities.
 * Any additional points introduced lie strictly between the two
 * adjacent hyperplanes and can therefore be integral.
 *
 *        ____			  _____
 *       /    ||\		 /     \
 *      /     || \		/       \
 *      \     ||  \	=>	\        \
 *       \    ||  /		 \       /
 *        \___||_/		  \_____/
 *
 * The test for a single pair of adjacent inequalities is important
 * for avoiding the combination of two basic maps like the following
 *
 *       /|
````
- **EN**: This block declares or defines routines around `wraps_free`, `isl_vec_free`, `isl_set_free`; emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `wraps_free`, `isl_vec_free`, `isl_set_free` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理; 保留用于解释意图、用法或算法背景的注释.

### Lines 1706-1731

````c
 *      / |
 *     /__|
 *         _____
 *         |   |
 *         |   |
 *         |___|
 *
 * If there are some cut constraints on one side, then we may
 * still be able to fuse the two basic maps, but we need to perform
 * some additional checks in is_adj_ineq_extension.
 */
static enum isl_change check_adj_ineq(int i, int j,
	struct isl_coalesce_info *info)
{
	int count_i, count_j;
	int cut_i, cut_j;

	count_i = count_ineq(&info[i], STATUS_ADJ_INEQ);
	count_j = count_ineq(&info[j], STATUS_ADJ_INEQ);

	if (count_i != 1 && count_j != 1)
		return isl_change_none;

	cut_i = any_eq(&info[i], STATUS_CUT) || any_ineq(&info[i], STATUS_CUT);
	cut_j = any_eq(&info[j], STATUS_CUT) || any_ineq(&info[j], STATUS_CUT);

````
- **EN**: This block declares or references types such as `isl_coalesce_info`; declares or defines routines around `check_adj_ineq`, `count_ineq`, `any_eq`; contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_coalesce_info`; 声明或定义与 `check_adj_ineq`, `count_ineq`, `any_eq` 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 1732-1763

````c
	if (!cut_i && !cut_j && count_i == 1 && count_j == 1)
		return fuse(i, j, info, NULL, 0, 0);

	if (count_i == 1 && !cut_i)
		return is_adj_ineq_extension(i, j, info);

	if (count_j == 1 && !cut_j)
		return is_adj_ineq_extension(j, i, info);

	return isl_change_none;
}

/* Given a basic set i with a constraint k that is adjacent to
 * basic set j, check if we can wrap
 * both the facet corresponding to k (if "wrap_facet" is set) and basic map j
 * (always) around their ridges to include the other set.
 * If so, replace the pair of basic sets by their union.
 *
 * All constraints of i (except k) are assumed to be valid or
 * cut constraints for j.
 * Wrapping the cut constraints to include basic map j may result
 * in constraints that are no longer valid of basic map i
 * we have to check that the resulting wrapping constraints are valid for i.
 * If "wrap_facet" is not set, then all constraints of i (except k)
 * are assumed to be valid for j.
 *        ____			  _____
 *       /    | 		 /     \
 *      /     ||  		/      |
 *      \     ||   	=>	\      |
 *       \    ||		 \     |
 *        \___||		  \____|
 *
````
- **EN**: This block declares or defines routines around `k`, `i`; contains control flow with 3 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `k`, `i` 相关的例程; 包含控制流结构：3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 1764-1790

````c
 */
static enum isl_change can_wrap_in_facet(int i, int j, int k,
	struct isl_coalesce_info *info, int wrap_facet)
{
	enum isl_change change = isl_change_none;
	struct isl_wraps wraps;
	isl_ctx *ctx;
	isl_mat *mat;
	struct isl_set *set_i = NULL;
	struct isl_set *set_j = NULL;
	struct isl_vec *bound = NULL;
	isl_size total = isl_basic_map_dim(info[i].bmap, isl_dim_all);

	if (total < 0)
		return isl_change_error;
	set_i = set_from_updated_bmap(&info[i]);
	set_j = set_from_updated_bmap(&info[j]);
	ctx = isl_basic_map_get_ctx(info[i].bmap);
	mat = isl_mat_alloc(ctx, 2 * (info[i].bmap->n_eq + info[j].bmap->n_eq) +
				    info[i].bmap->n_ineq + info[j].bmap->n_ineq,
				    1 + total);
	if (wraps_init(&wraps, mat, info, i, j) < 0)
		goto error;
	bound = isl_vec_alloc(ctx, 1 + total);
	if (!set_i || !set_j || !bound)
		goto error;

````
- **EN**: This block declares or references types such as `isl_coalesce_info`, `isl_wraps`, `isl_set`, `isl_vec`; defines enum values such as `isl_change`; declares or defines routines around `can_wrap_in_facet`, `isl_basic_map_dim`, `set_from_updated_bmap`, `isl_basic_map_get_ctx` (+2 more); contains control flow with 3 conditional check(s); and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_coalesce_info`, `isl_wraps`, `isl_set`, `isl_vec`; 定义枚举类型，例如 `isl_change`; 声明或定义与 `can_wrap_in_facet`, `isl_basic_map_dim`, `set_from_updated_bmap`, `isl_basic_map_get_ctx` (+2 more) 相关的例程; 包含控制流结构：3 处条件判断；并延续周边实现细节。

### Lines 1791-1818

````c
	isl_seq_cpy(bound->el, info[i].bmap->ineq[k], 1 + total);
	isl_int_add_ui(bound->el[0], bound->el[0], 1);
	isl_seq_normalize(ctx, bound->el, 1 + total);

	isl_seq_cpy(wraps.mat->row[0], bound->el, 1 + total);
	wraps.mat->n_row = 1;

	if (add_wraps(&wraps, &info[j], bound->el, set_i) < 0)
		goto error;
	if (wraps.failed)
		goto unbounded;

	if (wrap_facet) {
		if (add_wraps_around_facet(&wraps, &info[i], k,
					    bound->el, set_j) < 0)
			goto error;
		if (wraps.failed)
			goto unbounded;
	}

	change = fuse(i, j, info, wraps.mat, 0, 0);

unbounded:
	wraps_free(&wraps);

	isl_set_free(set_i);
	isl_set_free(set_j);

````
- **EN**: This block declares or defines routines around `isl_seq_cpy`, `isl_int_add_ui`, `isl_seq_normalize`, `fuse` (+2 more); contains control flow with 5 conditional check(s); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_seq_cpy`, `isl_int_add_ui`, `isl_seq_normalize`, `fuse` (+2 more) 相关的例程; 包含控制流结构：5 处条件判断; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1819-1850

````c
	isl_vec_free(bound);

	return change;
error:
	wraps_free(&wraps);
	isl_vec_free(bound);
	isl_set_free(set_i);
	isl_set_free(set_j);
	return isl_change_error;
}

/* Given a cut constraint t(x) >= 0 of basic map i, stored in row "w"
 * of wrap.mat, replace it by its relaxed version t(x) + 1 >= 0, and
 * add wrapping constraints to wrap.mat for all constraints
 * of basic map j that bound the part of basic map j that sticks out
 * of the cut constraint.
 * "set_i" is the underlying set of basic map i.
 * If any wrapping fails, then wraps->mat.n_row is reset to zero.
 *
 * In particular, we first intersect basic map j with t(x) + 1 = 0.
 * If the result is empty, then t(x) >= 0 was actually a valid constraint
 * (with respect to the integer points), so we add t(x) >= 0 instead.
 * Otherwise, we wrap the constraints of basic map j that are not
 * redundant in this intersection and that are not already valid
 * for basic map i over basic map i.
 * Note that it is sufficient to wrap the constraints to include
 * basic map i, because we will only wrap the constraints that do
 * not include basic map i already.  The wrapped constraint will
 * therefore be more relaxed compared to the original constraint.
 * Since the original constraint is valid for basic map j, so is
 * the wrapped constraint.
 */
````
- **EN**: This block declares or defines routines around `isl_vec_free`, `wraps_free`, `isl_set_free`, `t`; emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `isl_vec_free`, `wraps_free`, `isl_set_free`, `t` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理; 保留用于解释意图、用法或算法背景的注释.

### Lines 1851-1882

````c
static isl_stat wrap_in_facet(struct isl_wraps *wraps, int w,
	struct isl_coalesce_info *info_j, __isl_keep isl_set *set_i,
	struct isl_tab_undo *snap)
{
	isl_int_add_ui(wraps->mat->row[w][0], wraps->mat->row[w][0], 1);
	if (isl_tab_add_eq(info_j->tab, wraps->mat->row[w]) < 0)
		return isl_stat_error;
	if (isl_tab_detect_redundant(info_j->tab) < 0)
		return isl_stat_error;

	if (info_j->tab->empty)
		isl_int_sub_ui(wraps->mat->row[w][0], wraps->mat->row[w][0], 1);
	else if (add_wraps(wraps, info_j, wraps->mat->row[w], set_i) < 0)
		return isl_stat_error;

	if (isl_tab_rollback(info_j->tab, snap) < 0)
		return isl_stat_error;

	return isl_stat_ok;
}

/* Given a pair of basic maps i and j such that j sticks out
 * of i at n cut constraints, each time by at most one,
 * try to compute wrapping constraints and replace the two
 * basic maps by a single basic map.
 * The other constraints of i are assumed to be valid for j.
 * "set_i" is the underlying set of basic map i.
 * "wraps" has been initialized to be of the right size.
 *
 * For each cut constraint t(x) >= 0 of i, we add the relaxed version
 * t(x) + 1 >= 0, along with wrapping constraints for all constraints
 * of basic map j that bound the part of basic map j that sticks out
````
- **EN**: This block declares or references types such as `isl_coalesce_info`, `isl_tab_undo`; declares or defines routines around `wrap_in_facet`, `isl_int_add_ui`, `isl_int_sub_ui`, `t`; contains control flow with 5 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_coalesce_info`, `isl_tab_undo`; 声明或定义与 `wrap_in_facet`, `isl_int_add_ui`, `isl_int_sub_ui`, `t` 相关的例程; 包含控制流结构：5 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 1883-1916

````c
 * of the cut constraint.
 *
 * If any wrapping fails, i.e., if we cannot wrap to touch
 * the union, then we give up.
 * Otherwise, the pair of basic maps is replaced by their union.
 */
static enum isl_change try_wrap_in_facets(int i, int j,
	struct isl_coalesce_info *info, struct isl_wraps *wraps,
	__isl_keep isl_set *set_i)
{
	int k, l, w;
	isl_size total;
	struct isl_tab_undo *snap;

	total = isl_basic_map_dim(info[i].bmap, isl_dim_all);
	if (total < 0)
		return isl_change_error;

	snap = isl_tab_snap(info[j].tab);

	for (k = 0; k < info[i].bmap->n_eq; ++k) {
		for (l = 0; l < 2; ++l) {
			if (info[i].eq[2 * k + l] != STATUS_CUT)
				continue;
			w = wraps->mat->n_row++;
			if (l == 0)
				isl_seq_neg(wraps->mat->row[w],
					    info[i].bmap->eq[k], 1 + total);
			else
				isl_seq_cpy(wraps->mat->row[w],
					    info[i].bmap->eq[k], 1 + total);
			if (wrap_in_facet(wraps, w, &info[j], set_i, snap) < 0)
				return isl_change_error;

````
- **EN**: This block declares or references types such as `isl_coalesce_info`, `isl_tab_undo`; declares or defines routines around `try_wrap_in_facets`, `isl_basic_map_dim`, `isl_tab_snap`, `isl_seq_neg` (+1 more); contains control flow with 2 loop construct(s), 4 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_coalesce_info`, `isl_tab_undo`; 声明或定义与 `try_wrap_in_facets`, `isl_basic_map_dim`, `isl_tab_snap`, `isl_seq_neg` (+1 more) 相关的例程; 包含控制流结构：2 处循环、4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 1917-1948

````c
			if (wraps->failed)
				return isl_change_none;
		}
	}

	for (k = 0; k < info[i].bmap->n_ineq; ++k) {
		if (info[i].ineq[k] != STATUS_CUT)
			continue;
		w = wraps->mat->n_row++;
		isl_seq_cpy(wraps->mat->row[w],
			    info[i].bmap->ineq[k], 1 + total);
		if (wrap_in_facet(wraps, w, &info[j], set_i, snap) < 0)
			return isl_change_error;

		if (wraps->failed)
			return isl_change_none;
	}

	return fuse(i, j, info, wraps->mat, 0, 1);
}

/* Given a pair of basic maps i and j such that j sticks out
 * of i at n cut constraints, each time by at most one,
 * try to compute wrapping constraints and replace the two
 * basic maps by a single basic map.
 * The other constraints of i are assumed to be valid for j.
 *
 * The core computation is performed by try_wrap_in_facets.
 * This function simply extracts an underlying set representation
 * of basic map i and initializes the data structure for keeping
 * track of wrapping constraints.
 */
````
- **EN**: This block declares or defines routines around `isl_seq_cpy`; contains control flow with 1 loop construct(s), 4 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_seq_cpy` 相关的例程; 包含控制流结构：1 处循环、4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 1949-1975

````c
static enum isl_change wrap_in_facets(int i, int j, int n,
	struct isl_coalesce_info *info)
{
	enum isl_change change = isl_change_none;
	struct isl_wraps wraps;
	isl_ctx *ctx;
	isl_mat *mat;
	isl_set *set_i = NULL;
	isl_size total = isl_basic_map_dim(info[i].bmap, isl_dim_all);
	int max_wrap;

	if (total < 0)
		return isl_change_error;
	if (isl_tab_extend_cons(info[j].tab, 1) < 0)
		return isl_change_error;

	max_wrap = 1 + 2 * info[j].bmap->n_eq + info[j].bmap->n_ineq;
	max_wrap *= n;

	set_i = set_from_updated_bmap(&info[i]);
	ctx = isl_basic_map_get_ctx(info[i].bmap);
	mat = isl_mat_alloc(ctx, max_wrap, 1 + total);
	if (wraps_init(&wraps, mat, info, i, j) < 0)
		goto error;
	if (!set_i)
		goto error;

````
- **EN**: This block declares or references types such as `isl_coalesce_info`, `isl_wraps`; defines enum values such as `isl_change`; declares or defines routines around `wrap_in_facets`, `isl_basic_map_dim`, `set_from_updated_bmap`, `isl_basic_map_get_ctx` (+1 more); contains control flow with 4 conditional check(s); and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_coalesce_info`, `isl_wraps`; 定义枚举类型，例如 `isl_change`; 声明或定义与 `wrap_in_facets`, `isl_basic_map_dim`, `set_from_updated_bmap`, `isl_basic_map_get_ctx` (+1 more) 相关的例程; 包含控制流结构：4 处条件判断；并延续周边实现细节。

### Lines 1976-2001

````c
	change = try_wrap_in_facets(i, j, info, &wraps, set_i);

	wraps_free(&wraps);
	isl_set_free(set_i);

	return change;
error:
	wraps_free(&wraps);
	isl_set_free(set_i);
	return isl_change_error;
}

/* Return the effect of inequality "ineq" on the tableau "tab",
 * after relaxing the constant term of "ineq" by one.
 */
static enum isl_ineq_type type_of_relaxed(struct isl_tab *tab, isl_int *ineq)
{
	enum isl_ineq_type type;

	isl_int_add_ui(ineq[0], ineq[0], 1);
	type = isl_tab_ineq_type(tab, ineq);
	isl_int_sub_ui(ineq[0], ineq[0], 1);

	return type;
}

````
- **EN**: This block defines enum values such as `isl_ineq_type`; declares or defines routines around `try_wrap_in_facets`, `wraps_free`, `isl_set_free`, `type_of_relaxed` (+3 more); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 定义枚举类型，例如 `isl_ineq_type`; 声明或定义与 `try_wrap_in_facets`, `wraps_free`, `isl_set_free`, `type_of_relaxed` (+3 more) 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 2002-2033

````c
/* Given two basic sets i and j,
 * check if relaxing all the cut constraints of i by one turns
 * them into valid constraint for j and check if we can wrap in
 * the bits that are sticking out.
 * If so, replace the pair by their union.
 *
 * We first check if all relaxed cut inequalities of i are valid for j
 * and then try to wrap in the intersections of the relaxed cut inequalities
 * with j.
 *
 * During this wrapping, we consider the points of j that lie at a distance
 * of exactly 1 from i.  In particular, we ignore the points that lie in
 * between this lower-dimensional space and the basic map i.
 * We can therefore only apply this to integer maps.
 *        ____			  _____
 *       / ___|_		 /     \
 *      / |    |  		/      |
 *      \ |    |   	=>	\      |
 *       \|____|		 \     |
 *        \___| 		  \____/
 *
 *	 _____			 ______
 *	| ____|_		|      \
 *	| |     |		|       |
 *	| |	|	=>	|       |
 *	|_|     |		|       |
 *	  |_____|		 \______|
 *
 *	 _______
 *	|       |
 *	|  |\   |
 *	|  | \  |
````
- **EN**: This block preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 保留用于解释意图、用法或算法背景的注释.

### Lines 2034-2067

````c
 *	|  |  \ |
 *	|  |   \|
 *	|  |    \
 *	|  |_____\
 *	|       |
 *	|_______|
 *
 * Wrapping can fail if the result of wrapping one of the facets
 * around its edges does not produce any new facet constraint.
 * In particular, this happens when we try to wrap in unbounded sets.
 *
 *	 _______________________________________________________________________
 *	|
 *	|  ___
 *	| |   |
 *	|_|   |_________________________________________________________________
 *	  |___|
 *
 * The following is not an acceptable result of coalescing the above two
 * sets as it includes extra integer points.
 *	 _______________________________________________________________________
 *	|
 *	|     
 *	|      
 *	|
 *	 \______________________________________________________________________
 */
static enum isl_change can_wrap_in_set(int i, int j,
	struct isl_coalesce_info *info)
{
	int k, l;
	int n;
	isl_size total;

````
- **EN**: This block declares or references types such as `isl_coalesce_info`; declares or defines routines around `can_wrap_in_set`; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或引用类型，例如 `isl_coalesce_info`; 声明或定义与 `can_wrap_in_set` 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 2068-2100

````c
	if (ISL_F_ISSET(info[i].bmap, ISL_BASIC_MAP_RATIONAL) ||
	    ISL_F_ISSET(info[j].bmap, ISL_BASIC_MAP_RATIONAL))
		return isl_change_none;

	n = count_eq(&info[i], STATUS_CUT) + count_ineq(&info[i], STATUS_CUT);
	if (n == 0)
		return isl_change_none;

	total = isl_basic_map_dim(info[i].bmap, isl_dim_all);
	if (total < 0)
		return isl_change_error;
	for (k = 0; k < info[i].bmap->n_eq; ++k) {
		for (l = 0; l < 2; ++l) {
			enum isl_ineq_type type;

			if (info[i].eq[2 * k + l] != STATUS_CUT)
				continue;

			if (l == 0)
				isl_seq_neg(info[i].bmap->eq[k],
					    info[i].bmap->eq[k], 1 + total);
			type = type_of_relaxed(info[j].tab,
					    info[i].bmap->eq[k]);
			if (l == 0)
				isl_seq_neg(info[i].bmap->eq[k],
					    info[i].bmap->eq[k], 1 + total);
			if (type == isl_ineq_error)
				return isl_change_error;
			if (type != isl_ineq_redundant)
				return isl_change_none;
		}
	}

````
- **EN**: This block defines enum values such as `isl_ineq_type`; declares or defines routines around `ISL_F_ISSET`, `count_eq`, `isl_basic_map_dim`, `isl_seq_neg` (+1 more); contains control flow with 2 loop construct(s), 8 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 定义枚举类型，例如 `isl_ineq_type`; 声明或定义与 `ISL_F_ISSET`, `count_eq`, `isl_basic_map_dim`, `isl_seq_neg` (+1 more) 相关的例程; 包含控制流结构：2 处循环、8 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 2101-2128

````c
	for (k = 0; k < info[i].bmap->n_ineq; ++k) {
		enum isl_ineq_type type;

		if (info[i].ineq[k] != STATUS_CUT)
			continue;

		type = type_of_relaxed(info[j].tab, info[i].bmap->ineq[k]);
		if (type == isl_ineq_error)
			return isl_change_error;
		if (type != isl_ineq_redundant)
			return isl_change_none;
	}

	return wrap_in_facets(i, j, n, info);
}

/* Check if either i or j has only cut constraints that can
 * be used to wrap in (a facet of) the other basic set.
 * if so, replace the pair by their union.
 */
static enum isl_change check_wrap(int i, int j, struct isl_coalesce_info *info)
{
	enum isl_change change = isl_change_none;

	change = can_wrap_in_set(i, j, info);
	if (change != isl_change_none)
		return change;

````
- **EN**: This block defines enum values such as `isl_ineq_type`, `isl_change`; declares or defines routines around `type_of_relaxed`, `in`, `check_wrap`, `can_wrap_in_set`; contains control flow with 1 loop construct(s), 4 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 定义枚举类型，例如 `isl_ineq_type`, `isl_change`; 声明或定义与 `type_of_relaxed`, `in`, `check_wrap`, `can_wrap_in_set` 相关的例程; 包含控制流结构：1 处循环、4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 2129-2156

````c
	change = can_wrap_in_set(j, i, info);
	return change;
}

/* Check if all inequality constraints of "i" that cut "j" cease
 * to be cut constraints if they are relaxed by one.
 * If so, collect the cut constraints in "list".
 * The caller is responsible for allocating "list".
 */
static isl_bool all_cut_by_one(int i, int j, struct isl_coalesce_info *info,
	int *list)
{
	int l, n;

	n = 0;
	for (l = 0; l < info[i].bmap->n_ineq; ++l) {
		enum isl_ineq_type type;

		if (info[i].ineq[l] != STATUS_CUT)
			continue;
		type = type_of_relaxed(info[j].tab, info[i].bmap->ineq[l]);
		if (type == isl_ineq_error)
			return isl_bool_error;
		if (type != isl_ineq_redundant)
			return isl_bool_false;
		list[n++] = l;
	}

````
- **EN**: This block defines enum values such as `isl_ineq_type`; declares or defines routines around `can_wrap_in_set`, `all_cut_by_one`, `type_of_relaxed`; contains control flow with 1 loop construct(s), 3 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 定义枚举类型，例如 `isl_ineq_type`; 声明或定义与 `can_wrap_in_set`, `all_cut_by_one`, `type_of_relaxed` 相关的例程; 包含控制流结构：1 处循环、3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 2157-2185

````c
	return isl_bool_true;
}

/* Given two basic maps such that "j" has at least one equality constraint
 * that is adjacent to an inequality constraint of "i" and such that "i" has
 * exactly one inequality constraint that is adjacent to an equality
 * constraint of "j", check whether "i" can be extended to include "j" or
 * whether "j" can be wrapped into "i".
 * All remaining constraints of "i" and "j" are assumed to be valid
 * or cut constraints of the other basic map.
 * However, none of the equality constraints of "i" are cut constraints.
 *
 * If "i" has any "cut" inequality constraints, then check if relaxing
 * each of them by one is sufficient for them to become valid.
 * If so, check if the inequality constraint adjacent to an equality
 * constraint of "j" along with all these cut constraints
 * can be relaxed by one to contain exactly "j".
 * Otherwise, or if this fails, check if "j" can be wrapped into "i".
 */
static enum isl_change check_single_adj_eq(int i, int j,
	struct isl_coalesce_info *info)
{
	enum isl_change change = isl_change_none;
	int k;
	int n_cut;
	int *relax;
	isl_ctx *ctx;
	isl_bool try_relax;

````
- **EN**: This block declares or references types such as `isl_coalesce_info`; defines enum values such as `isl_change`; declares or defines routines around `check_single_adj_eq`; emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_coalesce_info`; 定义枚举类型，例如 `isl_change`; 声明或定义与 `check_single_adj_eq` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 2186-2211

````c
	n_cut = count_ineq(&info[i], STATUS_CUT);

	k = find_ineq(&info[i], STATUS_ADJ_EQ);

	if (n_cut > 0) {
		ctx = isl_basic_map_get_ctx(info[i].bmap);
		relax = isl_calloc_array(ctx, int, 1 + n_cut);
		if (!relax)
			return isl_change_error;
		relax[0] = k;
		try_relax = all_cut_by_one(i, j, info, relax + 1);
		if (try_relax < 0)
			change = isl_change_error;
	} else {
		try_relax = isl_bool_true;
		relax = &k;
	}
	if (try_relax && change == isl_change_none)
		change = is_relaxed_extension(i, j, 1 + n_cut, relax, info);
	if (n_cut > 0)
		free(relax);
	if (change != isl_change_none)
		return change;

	change = can_wrap_in_facet(i, j, k, info, n_cut > 0);

````
- **EN**: This block declares or defines routines around `count_ineq`, `find_ineq`, `isl_basic_map_get_ctx`, `isl_calloc_array` (+4 more); contains control flow with 6 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `count_ineq`, `find_ineq`, `isl_basic_map_get_ctx`, `isl_calloc_array` (+4 more) 相关的例程; 包含控制流结构：6 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 2212-2237

````c
	return change;
}

/* At least one of the basic maps has an equality that is adjacent
 * to an inequality.  Make sure that only one of the basic maps has
 * such an equality and that the other basic map has exactly one
 * inequality adjacent to an equality.
 * If the other basic map does not have such an inequality, then
 * check if all its constraints are either valid or cut constraints
 * and, if so, try wrapping in the first map into the second.
 * Otherwise, try to extend one basic map with the other or
 * wrap one basic map in the other.
 */
static enum isl_change check_adj_eq(int i, int j,
	struct isl_coalesce_info *info)
{
	if (any_eq(&info[i], STATUS_ADJ_INEQ) &&
	    any_eq(&info[j], STATUS_ADJ_INEQ))
		/* ADJ EQ TOO MANY */
		return isl_change_none;

	if (any_eq(&info[i], STATUS_ADJ_INEQ))
		return check_adj_eq(j, i, info);

	/* j has an equality adjacent to an inequality in i */

````
- **EN**: This block declares or references types such as `isl_coalesce_info`; declares or defines routines around `check_adj_eq`, `any_eq`; contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_coalesce_info`; 声明或定义与 `check_adj_eq`, `any_eq` 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 2238-2270

````c
	if (count_ineq(&info[i], STATUS_ADJ_EQ) != 1) {
		if (all_valid_or_cut(&info[i]))
			return can_wrap_in_set(i, j, info);
		return isl_change_none;
	}
	if (any_eq(&info[i], STATUS_CUT))
		return isl_change_none;
	if (any_ineq(&info[j], STATUS_ADJ_EQ) ||
	    any_ineq(&info[i], STATUS_ADJ_INEQ) ||
	    any_ineq(&info[j], STATUS_ADJ_INEQ))
		/* ADJ EQ TOO MANY */
		return isl_change_none;

	return check_single_adj_eq(i, j, info);
}

/* Disjunct "j" lies on a hyperplane that is adjacent to disjunct "i".
 * In particular, disjunct "i" has an inequality constraint that is adjacent
 * to a (combination of) equality constraint(s) of disjunct "j",
 * but disjunct "j" has no explicit equality constraint adjacent
 * to an inequality constraint of disjunct "i".
 *
 * Disjunct "i" is already known not to have any equality constraints
 * that are adjacent to an equality or inequality constraint.
 * Check that, other than the inequality constraint mentioned above,
 * all other constraints of disjunct "i" are valid for disjunct "j".
 * If so, try and wrap in disjunct "j".
 */
static enum isl_change check_ineq_adj_eq(int i, int j,
	struct isl_coalesce_info *info)
{
	int k;

````
- **EN**: This block declares or references types such as `isl_coalesce_info`; declares or defines routines around `any_ineq`, `a`, `check_ineq_adj_eq`; contains control flow with 4 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_coalesce_info`; 声明或定义与 `any_ineq`, `a`, `check_ineq_adj_eq` 相关的例程; 包含控制流结构：4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 2271-2302

````c
	if (any_eq(&info[i], STATUS_CUT))
		return isl_change_none;
	if (any_ineq(&info[i], STATUS_CUT))
		return isl_change_none;
	if (any_ineq(&info[i], STATUS_ADJ_INEQ))
		return isl_change_none;
	if (count_ineq(&info[i], STATUS_ADJ_EQ) != 1)
		return isl_change_none;

	k = find_ineq(&info[i], STATUS_ADJ_EQ);

	return can_wrap_in_facet(i, j, k, info, 0);
}

/* The two basic maps lie on adjacent hyperplanes.  In particular,
 * basic map "i" has an equality that lies parallel to basic map "j".
 * Check if we can wrap the facets around the parallel hyperplanes
 * to include the other set.
 *
 * We perform basically the same operations as can_wrap_in_facet,
 * except that we don't need to select a facet of one of the sets.
 *				_
 *	\\			\\
 *	 \\		=>	 \\
 *	  \			  \|
 *
 * If there is more than one equality of "i" adjacent to an equality of "j",
 * then the result will satisfy one or more equalities that are a linear
 * combination of these equalities.  These will be encoded as pairs
 * of inequalities in the wrapping constraints and need to be made
 * explicit.
 */
````
- **EN**: This block declares or defines routines around `find_ineq`; contains control flow with 4 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `find_ineq` 相关的例程; 包含控制流结构：4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 2303-2335

````c
static enum isl_change check_eq_adj_eq(int i, int j,
	struct isl_coalesce_info *info)
{
	int k;
	enum isl_change change = isl_change_none;
	int detect_equalities = 0;
	struct isl_wraps wraps;
	isl_ctx *ctx;
	isl_mat *mat;
	struct isl_set *set_i = NULL;
	struct isl_set *set_j = NULL;
	struct isl_vec *bound = NULL;
	isl_size total = isl_basic_map_dim(info[i].bmap, isl_dim_all);

	if (total < 0)
		return isl_change_error;
	if (count_eq(&info[i], STATUS_ADJ_EQ) != 1)
		detect_equalities = 1;

	k = find_eq(&info[i], STATUS_ADJ_EQ);

	set_i = set_from_updated_bmap(&info[i]);
	set_j = set_from_updated_bmap(&info[j]);
	ctx = isl_basic_map_get_ctx(info[i].bmap);
	mat = isl_mat_alloc(ctx, 2 * (info[i].bmap->n_eq + info[j].bmap->n_eq) +
				    info[i].bmap->n_ineq + info[j].bmap->n_ineq,
				    1 + total);
	if (wraps_init(&wraps, mat, info, i, j) < 0)
		goto error;
	bound = isl_vec_alloc(ctx, 1 + total);
	if (!set_i || !set_j || !bound)
		goto error;

````
- **EN**: This block declares or references types such as `isl_coalesce_info`, `isl_wraps`, `isl_set`, `isl_vec`; defines enum values such as `isl_change`; declares or defines routines around `check_eq_adj_eq`, `isl_basic_map_dim`, `find_eq`, `set_from_updated_bmap` (+3 more); contains control flow with 4 conditional check(s); and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_coalesce_info`, `isl_wraps`, `isl_set`, `isl_vec`; 定义枚举类型，例如 `isl_change`; 声明或定义与 `check_eq_adj_eq`, `isl_basic_map_dim`, `find_eq`, `set_from_updated_bmap` (+3 more) 相关的例程; 包含控制流结构：4 处条件判断；并延续周边实现细节。

### Lines 2336-2362

````c
	if (k % 2 == 0)
		isl_seq_neg(bound->el, info[i].bmap->eq[k / 2], 1 + total);
	else
		isl_seq_cpy(bound->el, info[i].bmap->eq[k / 2], 1 + total);
	isl_int_add_ui(bound->el[0], bound->el[0], 1);

	isl_seq_cpy(wraps.mat->row[0], bound->el, 1 + total);
	wraps.mat->n_row = 1;

	if (add_wraps(&wraps, &info[j], bound->el, set_i) < 0)
		goto error;
	if (wraps.failed)
		goto unbounded;

	isl_int_sub_ui(bound->el[0], bound->el[0], 1);
	isl_seq_neg(bound->el, bound->el, 1 + total);

	isl_seq_cpy(wraps.mat->row[wraps.mat->n_row], bound->el, 1 + total);
	wraps.mat->n_row++;

	if (add_wraps(&wraps, &info[i], bound->el, set_j) < 0)
		goto error;
	if (wraps.failed)
		goto unbounded;

	change = fuse(i, j, info, wraps.mat, detect_equalities, 0);

````
- **EN**: This block declares or defines routines around `isl_seq_neg`, `isl_seq_cpy`, `isl_int_add_ui`, `isl_int_sub_ui` (+1 more); contains control flow with 5 conditional check(s); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_seq_neg`, `isl_seq_cpy`, `isl_int_add_ui`, `isl_int_sub_ui` (+1 more) 相关的例程; 包含控制流结构：5 处条件判断; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 2363-2394

````c
	if (0) {
error:		change = isl_change_error;
	}
unbounded:

	wraps_free(&wraps);
	isl_set_free(set_i);
	isl_set_free(set_j);
	isl_vec_free(bound);

	return change;
}

/* Initialize the "eq" and "ineq" fields of "info".
 */
static void init_status(struct isl_coalesce_info *info)
{
	info->eq = info->ineq = NULL;
}

/* Set info->eq to the positions of the equalities of info->bmap
 * with respect to the basic map represented by "tab".
 * If info->eq has already been computed, then do not compute it again.
 */
static void set_eq_status_in(struct isl_coalesce_info *info,
	struct isl_tab *tab)
{
	if (info->eq)
		return;
	info->eq = eq_status_in(info->bmap, tab);
}

````
- **EN**: This block declares or references types such as `isl_tab`; declares or defines routines around `wraps_free`, `isl_set_free`, `isl_vec_free`, `init_status` (+2 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_tab`; 声明或定义与 `wraps_free`, `isl_set_free`, `isl_vec_free`, `init_status` (+2 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 2395-2426

````c
/* Set info->ineq to the positions of the inequalities of info->bmap
 * with respect to the basic map represented by "tab".
 * If info->ineq has already been computed, then do not compute it again.
 */
static void set_ineq_status_in(struct isl_coalesce_info *info,
	struct isl_tab *tab)
{
	if (info->ineq)
		return;
	info->ineq = ineq_status_in(info->bmap, info->tab, tab);
}

/* Free the memory allocated by the "eq" and "ineq" fields of "info".
 * This function assumes that init_status has been called on "info" first,
 * after which the "eq" and "ineq" fields may or may not have been
 * assigned a newly allocated array.
 */
static void clear_status(struct isl_coalesce_info *info)
{
	free(info->eq);
	free(info->ineq);
}

/* Are all inequality constraints of the basic map represented by "info"
 * valid for the other basic map, except for a single constraint
 * that is adjacent to an inequality constraint of the other basic map?
 */
static int all_ineq_valid_or_single_adj_ineq(struct isl_coalesce_info *info)
{
	int i;
	int k = -1;

````
- **EN**: This block declares or references types such as `isl_tab`; declares or defines routines around `set_ineq_status_in`, `ineq_status_in`, `clear_status`, `free` (+1 more); contains control flow with 1 conditional check(s); uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_tab`; 声明或定义与 `set_ineq_status_in`, `ineq_status_in`, `clear_status`, `free` (+1 more) 相关的例程; 包含控制流结构：1 处条件判断; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 2427-2458

````c
	for (i = 0; i < info->bmap->n_ineq; ++i) {
		if (info->ineq[i] == STATUS_REDUNDANT)
			continue;
		if (info->ineq[i] == STATUS_VALID)
			continue;
		if (info->ineq[i] != STATUS_ADJ_INEQ)
			return 0;
		if (k != -1)
			return 0;
		k = i;
	}

	return k != -1;
}

/* Basic map "i" has one or more equality constraints that separate it
 * from basic map "j".  Check if it happens to be an extension
 * of basic map "j".
 * In particular, check that all constraints of "j" are valid for "i",
 * except for one inequality constraint that is adjacent
 * to an inequality constraints of "i".
 * If so, check for "i" being an extension of "j" by calling
 * is_adj_ineq_extension.
 *
 * Clean up the memory allocated for keeping track of the status
 * of the constraints before returning.
 */
static enum isl_change separating_equality(int i, int j,
	struct isl_coalesce_info *info)
{
	enum isl_change change = isl_change_none;

````
- **EN**: This block declares or references types such as `isl_coalesce_info`; defines enum values such as `isl_change`; declares or defines routines around `separating_equality`; contains control flow with 1 loop construct(s), 4 conditional check(s); and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_coalesce_info`; 定义枚举类型，例如 `isl_change`; 声明或定义与 `separating_equality` 相关的例程; 包含控制流结构：1 处循环、4 处条件判断；并延续周边实现细节。

### Lines 2459-2490

````c
	if (all(info[j].eq, 2 * info[j].bmap->n_eq, STATUS_VALID) &&
	    all_ineq_valid_or_single_adj_ineq(&info[j]))
		change = is_adj_ineq_extension(j, i, info);

	clear_status(&info[i]);
	clear_status(&info[j]);
	return change;
}

/* Check if the union of the given pair of basic maps
 * can be represented by a single basic map.
 * If so, replace the pair by the single basic map and return
 * isl_change_drop_first, isl_change_drop_second or isl_change_fuse.
 * Otherwise, return isl_change_none.
 * The two basic maps are assumed to live in the same local space.
 * The "eq" and "ineq" fields of info[i] and info[j] are assumed
 * to have been initialized by the caller, either to NULL or
 * to valid information.
 *
 * We first check the effect of each constraint of one basic map
 * on the other basic map.
 * The constraint may be
 *	redundant	the constraint is redundant in its own
 *			basic map and should be ignore and removed
 *			in the end
 *	valid		all (integer) points of the other basic map
 *			satisfy the constraint
 *	separate	no (integer) point of the other basic map
 *			satisfies the constraint
 *	cut		some but not all points of the other basic map
 *			satisfy the constraint
 *	adj_eq		the given constraint is adjacent (on the outside)
````
- **EN**: This block declares or defines routines around `all_ineq_valid_or_single_adj_ineq`, `is_adj_ineq_extension`, `clear_status`, `all` (+2 more); contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `all_ineq_valid_or_single_adj_ineq`, `is_adj_ineq_extension`, `clear_status`, `all` (+2 more) 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 2491-2522

````c
 *			to an equality of the other basic map
 *	adj_ineq	the given constraint is adjacent (on the outside)
 *			to an inequality of the other basic map
 *
 * We consider seven cases in which we can replace the pair by a single
 * basic map.  We ignore all "redundant" constraints.
 *
 *	1. all constraints of one basic map are valid
 *		=> the other basic map is a subset and can be removed
 *
 *	2. all constraints of both basic maps are either "valid" or "cut"
 *	   and the facets corresponding to the "cut" constraints
 *	   of one of the basic maps lies entirely inside the other basic map
 *		=> the pair can be replaced by a basic map consisting
 *		   of the valid constraints in both basic maps
 *
 *	3. there is a single pair of adjacent inequalities
 *	   (all other constraints are "valid")
 *		=> the pair can be replaced by a basic map consisting
 *		   of the valid constraints in both basic maps
 *
 *	4. one basic map has a single adjacent inequality, while the other
 *	   constraints are "valid".  The other basic map has some
 *	   "cut" constraints, but replacing the adjacent inequality by
 *	   its opposite and adding the valid constraints of the other
 *	   basic map results in a subset of the other basic map
 *		=> the pair can be replaced by a basic map consisting
 *		   of the valid constraints in both basic maps
 *
 *	5. there is a single adjacent pair of an inequality and an equality,
 *	   the other constraints of the basic map containing the inequality are
 *	   "valid".  Moreover, if the inequality the basic map is relaxed
````
- **EN**: This block declares or defines routines around `adjacent`.
- **CN**: 该代码块 声明或定义与 `adjacent` 相关的例程.

### Lines 2523-2558

````c
 *	   and then turned into an equality, then resulting facet lies
 *	   entirely inside the other basic map
 *		=> the pair can be replaced by the basic map containing
 *		   the inequality, with the inequality relaxed.
 *
 *	6. there is a single inequality adjacent to an equality,
 *	   the other constraints of the basic map containing the inequality are
 *	   "valid".  Moreover, the facets corresponding to both
 *	   the inequality and the equality can be wrapped around their
 *	   ridges to include the other basic map
 *		=> the pair can be replaced by a basic map consisting
 *		   of the valid constraints in both basic maps together
 *		   with all wrapping constraints
 *
 *	7. one of the basic maps extends beyond the other by at most one.
 *	   Moreover, the facets corresponding to the cut constraints and
 *	   the pieces of the other basic map at offset one from these cut
 *	   constraints can be wrapped around their ridges to include
 *	   the union of the two basic maps
 *		=> the pair can be replaced by a basic map consisting
 *		   of the valid constraints in both basic maps together
 *		   with all wrapping constraints
 *
 *	8. the two basic maps live in adjacent hyperplanes.  In principle
 *	   such sets can always be combined through wrapping, but we impose
 *	   that there is only one such pair, to avoid overeager coalescing.
 *
 * Throughout the computation, we maintain a collection of tableaus
 * corresponding to the basic maps.  When the basic maps are dropped
 * or combined, the tableaus are modified accordingly.
 */
static enum isl_change coalesce_local_pair_reuse(int i, int j,
	struct isl_coalesce_info *info)
{
	enum isl_change change = isl_change_none;

````
- **EN**: This block declares or references types such as `isl_coalesce_info`; defines enum values such as `isl_change`; declares or defines routines around `coalesce_local_pair_reuse`; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或引用类型，例如 `isl_coalesce_info`; 定义枚举类型，例如 `isl_change`; 声明或定义与 `coalesce_local_pair_reuse` 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 2559-2586

````c
	set_ineq_status_in(&info[i], info[j].tab);
	if (info[i].bmap->n_ineq && !info[i].ineq)
		goto error;
	if (any_ineq(&info[i], STATUS_ERROR))
		goto error;
	if (any_ineq(&info[i], STATUS_SEPARATE))
		goto done;

	set_ineq_status_in(&info[j], info[i].tab);
	if (info[j].bmap->n_ineq && !info[j].ineq)
		goto error;
	if (any_ineq(&info[j], STATUS_ERROR))
		goto error;
	if (any_ineq(&info[j], STATUS_SEPARATE))
		goto done;

	set_eq_status_in(&info[i], info[j].tab);
	if (info[i].bmap->n_eq && !info[i].eq)
		goto error;
	if (any_eq(&info[i], STATUS_ERROR))
		goto error;

	set_eq_status_in(&info[j], info[i].tab);
	if (info[j].bmap->n_eq && !info[j].eq)
		goto error;
	if (any_eq(&info[j], STATUS_ERROR))
		goto error;

````
- **EN**: This block declares or defines routines around `set_ineq_status_in`, `set_eq_status_in`; contains control flow with 10 conditional check(s).
- **CN**: 该代码块 声明或定义与 `set_ineq_status_in`, `set_eq_status_in` 相关的例程; 包含控制流结构：10 处条件判断.

### Lines 2587-2621

````c
	if (any_eq(&info[i], STATUS_SEPARATE))
		return separating_equality(i, j, info);
	if (any_eq(&info[j], STATUS_SEPARATE))
		return separating_equality(j, i, info);

	if (all(info[i].eq, 2 * info[i].bmap->n_eq, STATUS_VALID) &&
	    all(info[i].ineq, info[i].bmap->n_ineq, STATUS_VALID)) {
		drop(&info[j]);
		change = isl_change_drop_second;
	} else if (all(info[j].eq, 2 * info[j].bmap->n_eq, STATUS_VALID) &&
		   all(info[j].ineq, info[j].bmap->n_ineq, STATUS_VALID)) {
		drop(&info[i]);
		change = isl_change_drop_first;
	} else if (any_eq(&info[i], STATUS_ADJ_EQ)) {
		change = check_eq_adj_eq(i, j, info);
	} else if (any_eq(&info[j], STATUS_ADJ_EQ)) {
		change = check_eq_adj_eq(j, i, info);
	} else if (any_eq(&info[i], STATUS_ADJ_INEQ) ||
		   any_eq(&info[j], STATUS_ADJ_INEQ)) {
		change = check_adj_eq(i, j, info);
	} else if (any_ineq(&info[i], STATUS_ADJ_EQ)) {
		change = check_ineq_adj_eq(i, j, info);
	} else if (any_ineq(&info[j], STATUS_ADJ_EQ)) {
		change = check_ineq_adj_eq(j, i, info);
	} else if (any_ineq(&info[i], STATUS_ADJ_INEQ) ||
		   any_ineq(&info[j], STATUS_ADJ_INEQ)) {
		change = check_adj_ineq(i, j, info);
	} else {
		if (!any_eq(&info[i], STATUS_CUT) &&
		    !any_eq(&info[j], STATUS_CUT))
			change = check_facets(i, j, info);
		if (change == isl_change_none)
			change = check_wrap(i, j, info);
	}

````
- **EN**: This block declares or defines routines around `all`, `drop`, `check_eq_adj_eq`, `any_eq` (+6 more); contains control flow with 12 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `all`, `drop`, `check_eq_adj_eq`, `any_eq` (+6 more) 相关的例程; 包含控制流结构：12 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 2622-2653

````c
done:
	clear_status(&info[i]);
	clear_status(&info[j]);
	return change;
error:
	clear_status(&info[i]);
	clear_status(&info[j]);
	return isl_change_error;
}

/* Check if the union of the given pair of basic maps
 * can be represented by a single basic map.
 * If so, replace the pair by the single basic map and return
 * isl_change_drop_first, isl_change_drop_second or isl_change_fuse.
 * Otherwise, return isl_change_none.
 * The two basic maps are assumed to live in the same local space.
 */
static enum isl_change coalesce_local_pair(int i, int j,
	struct isl_coalesce_info *info)
{
	init_status(&info[i]);
	init_status(&info[j]);
	return coalesce_local_pair_reuse(i, j, info);
}

/* Shift the integer division at position "div" of the basic map
 * represented by "info" by "shift".
 *
 * That is, if the integer division has the form
 *
 *	floor(f(x)/d)
 *
````
- **EN**: This block declares or references types such as `isl_coalesce_info`; declares or defines routines around `clear_status`, `coalesce_local_pair`, `init_status`, `floor`; emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_coalesce_info`; 声明或定义与 `clear_status`, `coalesce_local_pair`, `init_status`, `floor` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 2654-2685

````c
 * then replace it by
 *
 *	floor((f(x) + shift * d)/d) - shift
 */
static isl_stat shift_div(struct isl_coalesce_info *info, int div,
	isl_int shift)
{
	isl_size total, n_div;

	info->bmap = isl_basic_map_shift_div(info->bmap, div, 0, shift);
	if (!info->bmap)
		return isl_stat_error;

	total = isl_basic_map_dim(info->bmap, isl_dim_all);
	n_div = isl_basic_map_dim(info->bmap, isl_dim_div);
	if (total < 0 || n_div < 0)
		return isl_stat_error;
	total -= n_div;
	if (isl_tab_shift_var(info->tab, total + div, shift) < 0)
		return isl_stat_error;

	return isl_stat_ok;
}

/* If the integer division at position "div" is defined by an equality,
 * i.e., a stride constraint, then change the integer division expression
 * to have a constant term equal to zero.
 *
 * Let the equality constraint be
 *
 *	c + f + m a = 0
 *
````
- **EN**: This block declares or defines routines around `floor`, `shift_div`, `isl_basic_map_shift_div`, `isl_basic_map_dim`; contains control flow with 3 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `floor`, `shift_div`, `isl_basic_map_shift_div`, `isl_basic_map_dim` 相关的例程; 包含控制流结构：3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 2686-2713

````c
 * The integer division expression is then typically of the form
 *
 *	a = floor((-f - c')/m)
 *
 * The integer division is first shifted by t = floor(c/m),
 * turning the equality constraint into
 *
 *	c - m floor(c/m) + f + m a' = 0
 *
 * i.e.,
 *
 *	(c mod m) + f + m a' = 0
 *
 * That is,
 *
 *	a' = (-f - (c mod m))/m = floor((-f)/m)
 *
 * because a' is an integer and 0 <= (c mod m) < m.
 * The constant term of a' can therefore be zeroed out,
 * but only if the integer division expression is of the expected form.
 */
static isl_stat normalize_stride_div(struct isl_coalesce_info *info, int div)
{
	isl_bool defined, valid;
	isl_stat r;
	isl_constraint *c;
	isl_int shift, stride;

````
- **EN**: This block declares or defines routines around `floor`, `normalize_stride_div`; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `floor`, `normalize_stride_div` 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 2714-2742

````c
	defined = isl_basic_map_has_defining_equality(info->bmap, isl_dim_div,
							div, &c);
	if (defined < 0)
		return isl_stat_error;
	if (!defined)
		return isl_stat_ok;
	if (!c)
		return isl_stat_error;
	valid = isl_constraint_is_div_equality(c, div);
	isl_int_init(shift);
	isl_int_init(stride);
	isl_constraint_get_constant(c, &shift);
	isl_constraint_get_coefficient(c, isl_dim_div, div, &stride);
	isl_int_fdiv_q(shift, shift, stride);
	r = shift_div(info, div, shift);
	isl_int_clear(stride);
	isl_int_clear(shift);
	isl_constraint_free(c);
	if (r < 0 || valid < 0)
		return isl_stat_error;
	if (!valid)
		return isl_stat_ok;
	info->bmap = isl_basic_map_set_div_expr_constant_num_si_inplace(
							    info->bmap, div, 0);
	if (!info->bmap)
		return isl_stat_error;
	return isl_stat_ok;
}

````
- **EN**: This block declares or defines routines around `isl_basic_map_has_defining_equality`, `isl_constraint_is_div_equality`, `isl_int_init`, `isl_constraint_get_constant` (+6 more); contains control flow with 6 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_basic_map_has_defining_equality`, `isl_constraint_is_div_equality`, `isl_int_init`, `isl_constraint_get_constant` (+6 more) 相关的例程; 包含控制流结构：6 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 2743-2774

````c
/* The basic maps represented by "info1" and "info2" are known
 * to have the same number of integer divisions.
 * Check if pairs of integer divisions are equal to each other
 * despite the fact that they differ by a rational constant.
 *
 * In particular, look for any pair of integer divisions that
 * only differ in their constant terms.
 * If either of these integer divisions is defined
 * by stride constraints, then modify it to have a zero constant term.
 * If both are defined by stride constraints then in the end they will have
 * the same (zero) constant term.
 */
static isl_stat harmonize_stride_divs(struct isl_coalesce_info *info1,
	struct isl_coalesce_info *info2)
{
	int i;
	isl_size n;

	n = isl_basic_map_dim(info1->bmap, isl_dim_div);
	if (n < 0)
		return isl_stat_error;
	for (i = 0; i < n; ++i) {
		isl_bool known, harmonize;

		known = isl_basic_map_div_is_known(info1->bmap, i);
		if (known >= 0 && known)
			known = isl_basic_map_div_is_known(info2->bmap, i);
		if (known < 0)
			return isl_stat_error;
		if (!known)
			continue;
		harmonize = isl_basic_map_equal_div_expr_except_constant(
````
- **EN**: This block declares or references types such as `isl_coalesce_info`; declares or defines routines around `same`, `harmonize_stride_divs`, `isl_basic_map_dim`, `isl_basic_map_div_is_known` (+1 more); contains control flow with 1 loop construct(s), 4 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_coalesce_info`; 声明或定义与 `same`, `harmonize_stride_divs`, `isl_basic_map_dim`, `isl_basic_map_div_is_known` (+1 more) 相关的例程; 包含控制流结构：1 处循环、4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 2775-2809

````c
					    info1->bmap, i, info2->bmap, i);
		if (harmonize < 0)
			return isl_stat_error;
		if (!harmonize)
			continue;
		if (normalize_stride_div(info1, i) < 0)
			return isl_stat_error;
		if (normalize_stride_div(info2, i) < 0)
			return isl_stat_error;
	}

	return isl_stat_ok;
}

/* If "shift" is an integer constant, then shift the integer division
 * at position "div" of the basic map represented by "info" by "shift".
 * If "shift" is not an integer constant, then do nothing.
 * If "shift" is equal to zero, then no shift needs to be performed either.
 *
 * That is, if the integer division has the form
 *
 *	floor(f(x)/d)
 *
 * then replace it by
 *
 *	floor((f(x) + shift * d)/d) - shift
 */
static isl_stat shift_if_cst_int(struct isl_coalesce_info *info, int div,
	__isl_keep isl_aff *shift)
{
	isl_bool cst;
	isl_stat r;
	isl_int d;
	isl_val *c;

````
- **EN**: This block declares or defines routines around `floor`, `shift_if_cst_int`; contains control flow with 4 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `floor`, `shift_if_cst_int` 相关的例程; 包含控制流结构：4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 2810-2841

````c
	cst = isl_aff_is_cst(shift);
	if (cst < 0 || !cst)
		return cst < 0 ? isl_stat_error : isl_stat_ok;

	c = isl_aff_get_constant_val(shift);
	cst = isl_val_is_int(c);
	if (cst >= 0 && cst)
		cst = isl_bool_not(isl_val_is_zero(c));
	if (cst < 0 || !cst) {
		isl_val_free(c);
		return cst < 0 ? isl_stat_error : isl_stat_ok;
	}

	isl_int_init(d);
	r = isl_val_get_num_isl_int(c, &d);
	if (r >= 0)
		r = shift_div(info, div, d);
	isl_int_clear(d);

	isl_val_free(c);

	return r;
}

/* Check if some of the divs in the basic map represented by "info1"
 * are shifts of the corresponding divs in the basic map represented
 * by "info2", taking into account the equality constraints "eq1" of "info1"
 * and "eq2" of "info2".  If so, align them with those of "info2".
 * "info1" and "info2" are assumed to have the same number
 * of integer divisions.
 *
 * An integer division is considered to be a shift of another integer
````
- **EN**: This block declares or defines routines around `isl_aff_is_cst`, `isl_aff_get_constant_val`, `isl_val_is_int`, `isl_bool_not` (+5 more); contains control flow with 4 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_aff_is_cst`, `isl_aff_get_constant_val`, `isl_val_is_int`, `isl_bool_not` (+5 more) 相关的例程; 包含控制流结构：4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 2842-2874

````c
 * division if, after simplification with respect to the equality
 * constraints of the other basic map, one is equal to the other
 * plus a constant.
 *
 * In particular, for each pair of integer divisions, if both are known,
 * have the same denominator and are not already equal to each other,
 * simplify each with respect to the equality constraints
 * of the other basic map.  If the difference is an integer constant,
 * then move this difference outside.
 * That is, if, after simplification, one integer division is of the form
 *
 *	floor((f(x) + c_1)/d)
 *
 * while the other is of the form
 *
 *	floor((f(x) + c_2)/d)
 *
 * and n = (c_2 - c_1)/d is an integer, then replace the first
 * integer division by
 *
 *	floor((f_1(x) + c_1 + n * d)/d) - n,
 *
 * where floor((f_1(x) + c_1 + n * d)/d) = floor((f2(x) + c_2)/d)
 * after simplification with respect to the equality constraints.
 */
static isl_stat harmonize_divs_with_hulls(struct isl_coalesce_info *info1,
	struct isl_coalesce_info *info2, __isl_keep isl_basic_set *eq1,
	__isl_keep isl_basic_set *eq2)
{
	int i;
	isl_size total;
	isl_local_space *ls1, *ls2;

````
- **EN**: This block declares or references types such as `isl_coalesce_info`; declares or defines routines around `floor`, `harmonize_divs_with_hulls`; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或引用类型，例如 `isl_coalesce_info`; 声明或定义与 `floor`, `harmonize_divs_with_hulls` 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 2875-2906

````c
	total = isl_basic_map_dim(info1->bmap, isl_dim_all);
	if (total < 0)
		return isl_stat_error;
	ls1 = isl_local_space_wrap(isl_basic_map_get_local_space(info1->bmap));
	ls2 = isl_local_space_wrap(isl_basic_map_get_local_space(info2->bmap));
	for (i = 0; i < info1->bmap->n_div; ++i) {
		isl_stat r;
		isl_aff *div1, *div2;

		if (!isl_local_space_div_is_known(ls1, i) ||
		    !isl_local_space_div_is_known(ls2, i))
			continue;
		if (isl_int_ne(info1->bmap->div[i][0], info2->bmap->div[i][0]))
			continue;
		if (isl_seq_eq(info1->bmap->div[i] + 1,
				info2->bmap->div[i] + 1, 1 + total))
			continue;
		div1 = isl_local_space_get_div(ls1, i);
		div2 = isl_local_space_get_div(ls2, i);
		div1 = isl_aff_substitute_equalities(div1,
						    isl_basic_set_copy(eq2));
		div2 = isl_aff_substitute_equalities(div2,
						    isl_basic_set_copy(eq1));
		div2 = isl_aff_sub(div2, div1);
		r = shift_if_cst_int(info1, i, div2);
		isl_aff_free(div2);
		if (r < 0)
			break;
	}
	isl_local_space_free(ls1);
	isl_local_space_free(ls2);

````
- **EN**: This block declares or defines routines around `isl_basic_map_dim`, `isl_local_space_wrap`, `isl_local_space_div_is_known`, `isl_local_space_get_div` (+6 more); contains control flow with 1 loop construct(s), 5 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_basic_map_dim`, `isl_local_space_wrap`, `isl_local_space_div_is_known`, `isl_local_space_get_div` (+6 more) 相关的例程; 包含控制流结构：1 处循环、5 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 2907-2942

````c
	if (i < info1->bmap->n_div)
		return isl_stat_error;
	return isl_stat_ok;
}

/* Check if some of the divs in the basic map represented by "info1"
 * are shifts of the corresponding divs in the basic map represented
 * by "info2".  If so, align them with those of "info2".
 * Only do this if "info1" and "info2" have the same number
 * of integer divisions.
 *
 * An integer division is considered to be a shift of another integer
 * division if, after simplification with respect to the equality
 * constraints of the other basic map, one is equal to the other
 * plus a constant.
 *
 * First check if pairs of integer divisions are equal to each other
 * despite the fact that they differ by a rational constant.
 * If so, try and arrange for them to have the same constant term.
 *
 * Then, extract the equality constraints and continue with
 * harmonize_divs_with_hulls.
 *
 * If the equality constraints of both basic maps are the same,
 * then there is no need to perform any shifting since
 * the coefficients of the integer divisions should have been
 * reduced in the same way.
 */
static isl_stat harmonize_divs(struct isl_coalesce_info *info1,
	struct isl_coalesce_info *info2)
{
	isl_bool equal;
	isl_basic_map *bmap1, *bmap2;
	isl_basic_set *eq1, *eq2;
	isl_stat r;

````
- **EN**: This block declares or references types such as `isl_coalesce_info`; declares or defines routines around `harmonize_divs`; contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_coalesce_info`; 声明或定义与 `harmonize_divs` 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 2943-2970

````c
	if (!info1->bmap || !info2->bmap)
		return isl_stat_error;

	if (info1->bmap->n_div != info2->bmap->n_div)
		return isl_stat_ok;
	if (info1->bmap->n_div == 0)
		return isl_stat_ok;

	if (harmonize_stride_divs(info1, info2) < 0)
		return isl_stat_error;

	bmap1 = isl_basic_map_copy(info1->bmap);
	bmap2 = isl_basic_map_copy(info2->bmap);
	eq1 = isl_basic_map_wrap(isl_basic_map_plain_affine_hull(bmap1));
	eq2 = isl_basic_map_wrap(isl_basic_map_plain_affine_hull(bmap2));
	equal = isl_basic_set_plain_is_equal(eq1, eq2);
	if (equal < 0)
		r = isl_stat_error;
	else if (equal)
		r = isl_stat_ok;
	else
		r = harmonize_divs_with_hulls(info1, info2, eq1, eq2);
	isl_basic_set_free(eq1);
	isl_basic_set_free(eq2);

	return r;
}

````
- **EN**: This block declares or defines routines around `isl_basic_map_copy`, `isl_basic_map_wrap`, `isl_basic_set_plain_is_equal`, `harmonize_divs_with_hulls` (+1 more); contains control flow with 6 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_basic_map_copy`, `isl_basic_map_wrap`, `isl_basic_set_plain_is_equal`, `harmonize_divs_with_hulls` (+1 more) 相关的例程; 包含控制流结构：6 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 2971-2997

````c
/* Do the two basic maps live in the same local space, i.e.,
 * do they have the same (known) divs?
 * If either basic map has any unknown divs, then we can only assume
 * that they do not live in the same local space.
 */
static isl_bool same_divs(__isl_keep isl_basic_map *bmap1,
	__isl_keep isl_basic_map *bmap2)
{
	int i;
	isl_bool known;
	isl_size total;

	if (!bmap1 || !bmap2)
		return isl_bool_error;
	if (bmap1->n_div != bmap2->n_div)
		return isl_bool_false;

	if (bmap1->n_div == 0)
		return isl_bool_true;

	known = isl_basic_map_divs_known(bmap1);
	if (known < 0 || !known)
		return known;
	known = isl_basic_map_divs_known(bmap2);
	if (known < 0 || !known)
		return known;

````
- **EN**: This block declares or defines routines around `same`, `same_divs`, `isl_basic_map_divs_known`; contains control flow with 5 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `same`, `same_divs`, `isl_basic_map_divs_known` 相关的例程; 包含控制流结构：5 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 2998-3023

````c
	total = isl_basic_map_dim(bmap1, isl_dim_all);
	if (total < 0)
		return isl_bool_error;
	for (i = 0; i < bmap1->n_div; ++i)
		if (!isl_seq_eq(bmap1->div[i], bmap2->div[i], 2 + total))
			return isl_bool_false;

	return isl_bool_true;
}

/* Assuming that "tab" contains the equality constraints and
 * the initial inequality constraints of "bmap", copy the remaining
 * inequality constraints of "bmap" to "Tab".
 */
static isl_stat copy_ineq(struct isl_tab *tab, __isl_keep isl_basic_map *bmap)
{
	int i, n_ineq;

	if (!bmap)
		return isl_stat_error;

	n_ineq = tab->n_con - tab->n_eq;
	for (i = n_ineq; i < bmap->n_ineq; ++i)
		if (isl_tab_add_ineq(tab, bmap->ineq[i]) < 0)
			return isl_stat_error;

````
- **EN**: This block declares or defines routines around `isl_basic_map_dim`, `copy_ineq`; contains control flow with 2 loop construct(s), 4 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_basic_map_dim`, `copy_ineq` 相关的例程; 包含控制流结构：2 处循环、4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 3024-3055

````c
	return isl_stat_ok;
}

/* Description of an integer division that is added
 * during an expansion.
 * "pos" is the position of the corresponding variable.
 * "cst" indicates whether this integer division has a fixed value.
 * "val" contains the fixed value, if the value is fixed.
 */
struct isl_expanded {
	int pos;
	isl_bool cst;
	isl_int val;
};

/* For each of the "n" integer division variables "expanded",
 * if the variable has a fixed value, then add two inequality
 * constraints expressing the fixed value.
 * Otherwise, add the corresponding div constraints.
 * The caller is responsible for removing the div constraints
 * that it added for all these "n" integer divisions.
 *
 * The div constraints and the pair of inequality constraints
 * forcing the fixed value cannot both be added for a given variable
 * as the combination may render some of the original constraints redundant.
 * These would then be ignored during the coalescing detection,
 * while they could remain in the fused result.
 *
 * The two added inequality constraints are
 *
 *	-a + v >= 0
 *	a - v >= 0
````
- **EN**: This block declares or references types such as `isl_expanded`; emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或引用类型，例如 `isl_expanded`; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理; 保留用于解释意图、用法或算法背景的注释.

### Lines 3056-3081

````c
 *
 * with "a" the variable and "v" its fixed value.
 * The facet corresponding to one of these two constraints is selected
 * in the tableau to ensure that the pair of inequality constraints
 * is treated as an equality constraint.
 * Such implicit equality constraints need to be turned
 * into explicit equality constraints to ensure both sides
 * of the equality constraints are taken into account.
 *
 * The information in info->ineq is thrown away because it was
 * computed in terms of div constraints, while some of those
 * have now been replaced by these pairs of inequality constraints.
 */
static isl_stat fix_constant_divs(struct isl_coalesce_info *info,
	int n, struct isl_expanded *expanded)
{
	unsigned o_div;
	int i;
	isl_vec *ineq;

	o_div = isl_basic_map_offset(info->bmap, isl_dim_div) - 1;
	ineq = isl_vec_alloc(isl_tab_get_ctx(info->tab), 1 + info->tab->n_var);
	if (!ineq)
		return isl_stat_error;
	isl_seq_clr(ineq->el + 1, info->tab->n_var);

````
- **EN**: This block declares or defines routines around `fix_constant_divs`, `isl_basic_map_offset`, `isl_vec_alloc`, `isl_seq_clr`; contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `fix_constant_divs`, `isl_basic_map_offset`, `isl_vec_alloc`, `isl_seq_clr` 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 3082-3107

````c
	for (i = 0; i < n; ++i) {
		if (!expanded[i].cst) {
			info->bmap = isl_basic_map_extend_constraints(
						info->bmap, 0, 2);
			info->bmap = isl_basic_map_add_div_constraints(
					info->bmap, expanded[i].pos - o_div);
		} else {
			isl_int_set_si(ineq->el[1 + expanded[i].pos], -1);
			isl_int_set(ineq->el[0], expanded[i].val);
			info->bmap = isl_basic_map_add_ineq(info->bmap,
								ineq->el);
			isl_int_set_si(ineq->el[1 + expanded[i].pos], 1);
			isl_int_neg(ineq->el[0], expanded[i].val);
			info->bmap = isl_basic_map_add_ineq(info->bmap,
								ineq->el);
			isl_int_set_si(ineq->el[1 + expanded[i].pos], 0);
		}
		if (copy_ineq(info->tab, info->bmap) < 0)
			break;
		if (expanded[i].cst &&
		    isl_tab_select_facet(info->tab, info->tab->n_con - 1) < 0)
			break;
	}

	isl_vec_free(ineq);

````
- **EN**: This block declares or defines routines around `isl_basic_map_extend_constraints`, `isl_basic_map_add_div_constraints`, `isl_int_set_si`, `isl_int_set` (+4 more); contains control flow with 1 loop construct(s), 3 conditional check(s); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_basic_map_extend_constraints`, `isl_basic_map_add_div_constraints`, `isl_int_set_si`, `isl_int_set` (+4 more) 相关的例程; 包含控制流结构：1 处循环、3 处条件判断; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 3108-3139

````c
	clear_status(info);
	init_status(info);

	info->bmap = isl_tab_make_equalities_explicit(info->tab, info->bmap);

	return i < n ? isl_stat_error : isl_stat_ok;
}

/* Insert the "n" integer division variables "expanded"
 * into info->tab and info->bmap and
 * update info->ineq with respect to the redundant constraints
 * in the resulting tableau.
 * "bmap" contains the result of this insertion in info->bmap,
 * while info->bmap is the original version
 * of "bmap", i.e., the one that corresponds to the current
 * state of info->tab.  The number of constraints in info->bmap
 * is assumed to be the same as the number of constraints
 * in info->tab.  This is required to be able to detect
 * the extra constraints in "bmap".
 *
 * In particular, introduce extra variables corresponding
 * to the extra integer divisions and add the div constraints
 * that were added to "bmap" after info->tab was created
 * from info->bmap.
 * Furthermore, check if these extra integer divisions happen
 * to attain a fixed integer value in info->tab.
 * If so, replace the corresponding div constraints by pairs
 * of inequality constraints that fix these
 * integer divisions to their single integer values.
 * Replace info->bmap by "bmap" to match the changes to info->tab.
 * info->ineq was computed without a tableau and therefore
 * does not take into account the redundant constraints
````
- **EN**: This block declares or defines routines around `clear_status`, `init_status`, `isl_tab_make_equalities_explicit`; emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `clear_status`, `init_status`, `isl_tab_make_equalities_explicit` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理; 保留用于解释意图、用法或算法背景的注释.

### Lines 3140-3165

````c
 * in the tableau.  Mark them here.
 * There is no need to check the newly added div constraints
 * since they cannot be redundant.
 * The redundancy check is not performed when constants have been discovered
 * since info->ineq is completely thrown away in this case.
 */
static isl_stat tab_insert_divs(struct isl_coalesce_info *info,
	int n, struct isl_expanded *expanded, __isl_take isl_basic_map *bmap)
{
	int i, n_ineq;
	unsigned n_eq;
	struct isl_tab_undo *snap;
	int any;

	if (!bmap)
		return isl_stat_error;
	if (info->bmap->n_eq + info->bmap->n_ineq != info->tab->n_con)
		isl_die(isl_basic_map_get_ctx(bmap), isl_error_internal,
			"original tableau does not correspond "
			"to original basic map", goto error);

	if (isl_tab_extend_vars(info->tab, n) < 0)
		goto error;
	if (isl_tab_extend_cons(info->tab, 2 * n) < 0)
		goto error;

````
- **EN**: This block declares or references types such as `isl_tab_undo`; declares or defines routines around `tab_insert_divs`, `isl_die`; contains control flow with 4 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_tab_undo`; 声明或定义与 `tab_insert_divs`, `isl_die` 相关的例程; 包含控制流结构：4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 3166-3197

````c
	for (i = 0; i < n; ++i) {
		if (isl_tab_insert_var(info->tab, expanded[i].pos) < 0)
			goto error;
	}

	snap = isl_tab_snap(info->tab);

	n_ineq = info->tab->n_con - info->tab->n_eq;
	if (copy_ineq(info->tab, bmap) < 0)
		goto error;

	isl_basic_map_free(info->bmap);
	info->bmap = bmap;

	any = 0;
	for (i = 0; i < n; ++i) {
		expanded[i].cst = isl_tab_is_constant(info->tab,
					    expanded[i].pos, &expanded[i].val);
		if (expanded[i].cst < 0)
			return isl_stat_error;
		if (expanded[i].cst)
			any = 1;
	}

	if (any) {
		if (isl_tab_rollback(info->tab, snap) < 0)
			return isl_stat_error;
		info->bmap = isl_basic_map_cow(info->bmap);
		info->bmap = isl_basic_map_free_inequality(info->bmap, 2 * n);
		if (!info->bmap)
			return isl_stat_error;

````
- **EN**: This block declares or defines routines around `isl_tab_snap`, `isl_basic_map_free`, `isl_tab_is_constant`, `isl_basic_map_cow` (+1 more); contains control flow with 2 loop construct(s), 7 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_tab_snap`, `isl_basic_map_free`, `isl_tab_is_constant`, `isl_basic_map_cow` (+1 more) 相关的例程; 包含控制流结构：2 处循环、7 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 3198-3229

````c
		return fix_constant_divs(info, n, expanded);
	}

	n_eq = info->bmap->n_eq;
	for (i = 0; i < n_ineq; ++i) {
		if (isl_tab_is_redundant(info->tab, n_eq + i))
			info->ineq[i] = STATUS_REDUNDANT;
	}

	return isl_stat_ok;
error:
	isl_basic_map_free(bmap);
	return isl_stat_error;
}

/* Expand info->tab and info->bmap in the same way "bmap" was expanded
 * in isl_basic_map_expand_divs using the expansion "exp" and
 * update info->ineq with respect to the redundant constraints
 * in the resulting tableau. info->bmap is the original version
 * of "bmap", i.e., the one that corresponds to the current
 * state of info->tab.  The number of constraints in info->bmap
 * is assumed to be the same as the number of constraints
 * in info->tab.  This is required to be able to detect
 * the extra constraints in "bmap".
 *
 * Extract the positions where extra local variables are introduced
 * from "exp" and call tab_insert_divs.
 */
static isl_stat expand_tab(struct isl_coalesce_info *info, int *exp,
	__isl_take isl_basic_map *bmap)
{
	isl_ctx *ctx;
````
- **EN**: This block declares or defines routines around `isl_basic_map_free`, `expand_tab`; contains control flow with 1 loop construct(s), 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_basic_map_free`, `expand_tab` 相关的例程; 包含控制流结构：1 处循环、1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 3230-3259

````c
	struct isl_expanded *expanded;
	int i, j, k, n;
	int extra_var;
	isl_size total, n_div;
	unsigned pos;
	isl_stat r;

	total = isl_basic_map_dim(bmap, isl_dim_all);
	n_div = isl_basic_map_dim(bmap, isl_dim_div);
	if (total < 0 || n_div < 0)
		return isl_stat_error;
	pos = total - n_div;
	extra_var = total - info->tab->n_var;
	n = n_div - extra_var;

	ctx = isl_basic_map_get_ctx(bmap);
	expanded = isl_calloc_array(ctx, struct isl_expanded, extra_var);
	if (extra_var && !expanded)
		goto error;

	i = 0;
	k = 0;
	for (j = 0; j < n_div; ++j) {
		if (i < n && exp[i] == j) {
			++i;
			continue;
		}
		expanded[k++].pos = pos + j;
	}

````
- **EN**: This block declares or references types such as `isl_expanded`; declares or defines routines around `isl_basic_map_dim`, `isl_basic_map_get_ctx`, `isl_calloc_array`; contains control flow with 1 loop construct(s), 3 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_expanded`; 声明或定义与 `isl_basic_map_dim`, `isl_basic_map_get_ctx`, `isl_calloc_array` 相关的例程; 包含控制流结构：1 处循环、3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 3260-3291

````c
	for (k = 0; k < extra_var; ++k)
		isl_int_init(expanded[k].val);

	r = tab_insert_divs(info, extra_var, expanded, bmap);

	for (k = 0; k < extra_var; ++k)
		isl_int_clear(expanded[k].val);
	free(expanded);

	return r;
error:
	isl_basic_map_free(bmap);
	return isl_stat_error;
}

/* Check if the union of the basic maps represented by info[i] and info[j]
 * can be represented by a single basic map,
 * after expanding the divs of info[i] to match those of info[j].
 * If so, replace the pair by the single basic map and return
 * isl_change_drop_first, isl_change_drop_second or isl_change_fuse.
 * Otherwise, return isl_change_none.
 *
 * The caller has already checked for info[j] being a subset of info[i].
 * If some of the divs of info[j] are unknown, then the expanded info[i]
 * will not have the corresponding div constraints.  The other patterns
 * therefore cannot apply.  Skip the computation in this case.
 *
 * The expansion is performed using the divs "div" and expansion "exp"
 * computed by the caller.
 * info[i].bmap has already been expanded and the result is passed in
 * as "bmap".
 * The "eq" and "ineq" fields of info[i] reflect the status of
````
- **EN**: This block declares or defines routines around `isl_int_init`, `tab_insert_divs`, `isl_int_clear`, `free` (+1 more); contains control flow with 2 loop construct(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_int_init`, `tab_insert_divs`, `isl_int_clear`, `free` (+1 more) 相关的例程; 包含控制流结构：2 处循环; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 3292-3320

````c
 * the constraints of the expanded "bmap" with respect to info[j].tab.
 * However, inequality constraints that are redundant in info[i].tab
 * have not yet been marked as such because no tableau was available.
 *
 * Replace info[i].bmap by "bmap" and expand info[i].tab as well,
 * updating info[i].ineq with respect to the redundant constraints.
 * Then try and coalesce the expanded info[i] with info[j],
 * reusing the information in info[i].eq and info[i].ineq.
 * If this does not result in any coalescing or if it results in info[j]
 * getting dropped (which should not happen in practice, since the case
 * of info[j] being a subset of info[i] has already been checked by
 * the caller), then revert info[i] to its original state.
 */
static enum isl_change coalesce_expand_tab_divs(__isl_take isl_basic_map *bmap,
	int i, int j, struct isl_coalesce_info *info, __isl_keep isl_mat *div,
	int *exp)
{
	isl_bool known;
	isl_basic_map *bmap_i;
	struct isl_tab_undo *snap;
	enum isl_change change = isl_change_none;

	known = isl_basic_map_divs_known(info[j].bmap);
	if (known < 0 || !known) {
		clear_status(&info[i]);
		isl_basic_map_free(bmap);
		return known < 0 ? isl_change_error : isl_change_none;
	}

````
- **EN**: This block declares or references types such as `isl_tab_undo`; defines enum values such as `isl_change`; declares or defines routines around `dropped`, `coalesce_expand_tab_divs`, `isl_basic_map_divs_known`, `clear_status` (+1 more); contains control flow with 1 conditional check(s); and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_tab_undo`; 定义枚举类型，例如 `isl_change`; 声明或定义与 `dropped`, `coalesce_expand_tab_divs`, `isl_basic_map_divs_known`, `clear_status` (+1 more) 相关的例程; 包含控制流结构：1 处条件判断；并延续周边实现细节。

### Lines 3321-3352

````c
	bmap_i = isl_basic_map_copy(info[i].bmap);
	snap = isl_tab_snap(info[i].tab);
	if (expand_tab(&info[i], exp, bmap) < 0)
		change = isl_change_error;

	init_status(&info[j]);
	if (change == isl_change_none)
		change = coalesce_local_pair_reuse(i, j, info);
	else
		clear_status(&info[i]);
	if (change != isl_change_none && change != isl_change_drop_second) {
		isl_basic_map_free(bmap_i);
	} else {
		isl_basic_map_free(info[i].bmap);
		info[i].bmap = bmap_i;

		if (isl_tab_rollback(info[i].tab, snap) < 0)
			change = isl_change_error;
	}

	return change;
}

/* Check if the union of "bmap" and the basic map represented by info[j]
 * can be represented by a single basic map,
 * after expanding the divs of "bmap" to match those of info[j].
 * If so, replace the pair by the single basic map and return
 * isl_change_drop_first, isl_change_drop_second or isl_change_fuse.
 * Otherwise, return isl_change_none.
 *
 * In particular, check if the expanded "bmap" contains the basic map
 * represented by the tableau info[j].tab.
````
- **EN**: This block declares or defines routines around `isl_basic_map_copy`, `isl_tab_snap`, `init_status`, `coalesce_local_pair_reuse` (+2 more); contains control flow with 4 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_basic_map_copy`, `isl_tab_snap`, `init_status`, `coalesce_local_pair_reuse` (+2 more) 相关的例程; 包含控制流结构：4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 3353-3380

````c
 * The expansion is performed using the divs "div" and expansion "exp"
 * computed by the caller.
 * Then we check if all constraints of the expanded "bmap" are valid for
 * info[j].tab.
 *
 * If "i" is not equal to -1, then "bmap" is equal to info[i].bmap.
 * In this case, the positions of the constraints of info[i].bmap
 * with respect to the basic map represented by info[j] are stored
 * in info[i].
 *
 * If the expanded "bmap" does not contain the basic map
 * represented by the tableau info[j].tab and if "i" is not -1,
 * i.e., if the original "bmap" is info[i].bmap, then expand info[i].tab
 * as well and check if that results in coalescing.
 */
static enum isl_change coalesce_with_expanded_divs(
	__isl_keep isl_basic_map *bmap, int i, int j,
	struct isl_coalesce_info *info, __isl_keep isl_mat *div, int *exp)
{
	enum isl_change change = isl_change_none;
	struct isl_coalesce_info info_local, *info_i;

	info_i = i >= 0 ? &info[i] : &info_local;
	init_status(info_i);
	bmap = isl_basic_map_copy(bmap);
	bmap = isl_basic_map_expand_divs(bmap, isl_mat_copy(div), exp);
	bmap = isl_basic_map_mark_final(bmap);

````
- **EN**: This block declares or references types such as `isl_coalesce_info`; defines enum values such as `isl_change`; declares or defines routines around `coalesce_with_expanded_divs`, `init_status`, `isl_basic_map_copy`, `isl_basic_map_expand_divs` (+1 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或引用类型，例如 `isl_coalesce_info`; 定义枚举类型，例如 `isl_change`; 声明或定义与 `coalesce_with_expanded_divs`, `init_status`, `isl_basic_map_copy`, `isl_basic_map_expand_divs` (+1 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 3381-3406

````c
	if (!bmap)
		goto error;

	info_local.bmap = bmap;
	info_i->eq = eq_status_in(bmap, info[j].tab);
	if (bmap->n_eq && !info_i->eq)
		goto error;
	if (any_eq(info_i, STATUS_ERROR))
		goto error;
	if (any_eq(info_i, STATUS_SEPARATE))
		goto done;

	info_i->ineq = ineq_status_in(bmap, NULL, info[j].tab);
	if (bmap->n_ineq && !info_i->ineq)
		goto error;
	if (any_ineq(info_i, STATUS_ERROR))
		goto error;
	if (any_ineq(info_i, STATUS_SEPARATE))
		goto done;

	if (all(info_i->eq, 2 * bmap->n_eq, STATUS_VALID) &&
	    all(info_i->ineq, bmap->n_ineq, STATUS_VALID)) {
		drop(&info[j]);
		change = isl_change_drop_second;
	}

````
- **EN**: This block declares or defines routines around `eq_status_in`, `ineq_status_in`, `all`, `drop`; contains control flow with 8 conditional check(s); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `eq_status_in`, `ineq_status_in`, `all`, `drop` 相关的例程; 包含控制流结构：8 处条件判断; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 3407-3438

````c
	if (change == isl_change_none && i != -1)
		return coalesce_expand_tab_divs(bmap, i, j, info, div, exp);

done:
	isl_basic_map_free(bmap);
	clear_status(info_i);
	return change;
error:
	isl_basic_map_free(bmap);
	clear_status(info_i);
	return isl_change_error;
}

/* Check if the union of "bmap_i" and the basic map represented by info[j]
 * can be represented by a single basic map,
 * after aligning the divs of "bmap_i" to match those of info[j].
 * If so, replace the pair by the single basic map and return
 * isl_change_drop_first, isl_change_drop_second or isl_change_fuse.
 * Otherwise, return isl_change_none.
 *
 * In particular, check if "bmap_i" contains the basic map represented by
 * info[j] after aligning the divs of "bmap_i" to those of info[j].
 * Note that this can only succeed if the number of divs of "bmap_i"
 * is smaller than (or equal to) the number of divs of info[j].
 *
 * We first check if the divs of "bmap_i" are all known and form a subset
 * of those of info[j].bmap.  If so, we pass control over to
 * coalesce_with_expanded_divs.
 *
 * If "i" is not equal to -1, then "bmap" is equal to info[i].bmap.
 */
static enum isl_change coalesce_after_aligning_divs(
````
- **EN**: This block declares or defines routines around `isl_basic_map_free`, `clear_status`, `than`, `coalesce_after_aligning_divs`; contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_basic_map_free`, `clear_status`, `than`, `coalesce_after_aligning_divs` 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 3439-3467

````c
	__isl_keep isl_basic_map *bmap_i, int i, int j,
	struct isl_coalesce_info *info)
{
	isl_bool known;
	isl_mat *div_i, *div_j, *div;
	int *exp1 = NULL;
	int *exp2 = NULL;
	isl_ctx *ctx;
	enum isl_change change;

	known = isl_basic_map_divs_known(bmap_i);
	if (known < 0)
		return isl_change_error;
	if (!known)
		return isl_change_none;

	ctx = isl_basic_map_get_ctx(bmap_i);

	div_i = isl_basic_map_get_divs(bmap_i);
	div_j = isl_basic_map_get_divs(info[j].bmap);

	if (!div_i || !div_j)
		goto error;

	exp1 = isl_alloc_array(ctx, int, div_i->n_row);
	exp2 = isl_alloc_array(ctx, int, div_j->n_row);
	if ((div_i->n_row && !exp1) || (div_j->n_row && !exp2))
		goto error;

````
- **EN**: This block declares or references types such as `isl_coalesce_info`; defines enum values such as `isl_change`; declares or defines routines around `isl_basic_map_divs_known`, `isl_basic_map_get_ctx`, `isl_basic_map_get_divs`, `isl_alloc_array`; contains control flow with 4 conditional check(s); and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_coalesce_info`; 定义枚举类型，例如 `isl_change`; 声明或定义与 `isl_basic_map_divs_known`, `isl_basic_map_get_ctx`, `isl_basic_map_get_divs`, `isl_alloc_array` 相关的例程; 包含控制流结构：4 处条件判断；并延续周边实现细节。

### Lines 3468-3494

````c
	div = isl_merge_divs(div_i, div_j, exp1, exp2);
	if (!div)
		goto error;

	if (div->n_row == div_j->n_row)
		change = coalesce_with_expanded_divs(bmap_i,
							i, j, info, div, exp1);
	else
		change = isl_change_none;

	isl_mat_free(div);

	isl_mat_free(div_i);
	isl_mat_free(div_j);

	free(exp2);
	free(exp1);

	return change;
error:
	isl_mat_free(div_i);
	isl_mat_free(div_j);
	free(exp1);
	free(exp2);
	return isl_change_error;
}

````
- **EN**: This block declares or defines routines around `isl_merge_divs`, `coalesce_with_expanded_divs`, `isl_mat_free`, `free`; contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_merge_divs`, `coalesce_with_expanded_divs`, `isl_mat_free`, `free` 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 3495-3523

````c
/* Check if basic map "j" is a subset of basic map "i" after
 * exploiting the extra equalities of "j" to simplify the divs of "i".
 * If so, remove basic map "j" and return isl_change_drop_second.
 *
 * If "j" does not have any equalities or if they are the same
 * as those of "i", then we cannot exploit them to simplify the divs.
 * Similarly, if there are no divs in "i", then they cannot be simplified.
 * If, on the other hand, the affine hulls of "i" and "j" do not intersect,
 * then "j" cannot be a subset of "i".
 *
 * Otherwise, we intersect "i" with the affine hull of "j" and then
 * check if "j" is a subset of the result after aligning the divs.
 * If so, then "j" is definitely a subset of "i" and can be removed.
 * Note that if after intersection with the affine hull of "j".
 * "i" still has more divs than "j", then there is no way we can
 * align the divs of "i" to those of "j".
 */
static enum isl_change coalesce_subset_with_equalities(int i, int j,
	struct isl_coalesce_info *info)
{
	isl_basic_map *hull_i, *hull_j, *bmap_i;
	int equal, empty;
	enum isl_change change;

	if (info[j].bmap->n_eq == 0)
		return isl_change_none;
	if (info[i].bmap->n_div == 0)
		return isl_change_none;

````
- **EN**: This block declares or references types such as `isl_coalesce_info`; defines enum values such as `isl_change`; declares or defines routines around `coalesce_subset_with_equalities`; contains control flow with 2 conditional check(s); and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_coalesce_info`; 定义枚举类型，例如 `isl_change`; 声明或定义与 `coalesce_subset_with_equalities` 相关的例程; 包含控制流结构：2 处条件判断；并延续周边实现细节。

### Lines 3524-3550

````c
	hull_i = isl_basic_map_copy(info[i].bmap);
	hull_i = isl_basic_map_plain_affine_hull(hull_i);
	hull_j = isl_basic_map_copy(info[j].bmap);
	hull_j = isl_basic_map_plain_affine_hull(hull_j);

	hull_j = isl_basic_map_intersect(hull_j, isl_basic_map_copy(hull_i));
	equal = isl_basic_map_plain_is_equal(hull_i, hull_j);
	empty = isl_basic_map_plain_is_empty(hull_j);
	isl_basic_map_free(hull_i);

	if (equal < 0 || equal || empty < 0 || empty) {
		isl_basic_map_free(hull_j);
		if (equal < 0 || empty < 0)
			return isl_change_error;
		return isl_change_none;
	}

	bmap_i = isl_basic_map_copy(info[i].bmap);
	bmap_i = isl_basic_map_intersect(bmap_i, hull_j);
	if (!bmap_i)
		return isl_change_error;

	if (bmap_i->n_div > info[j].bmap->n_div) {
		isl_basic_map_free(bmap_i);
		return isl_change_none;
	}

````
- **EN**: This block declares or defines routines around `isl_basic_map_copy`, `isl_basic_map_plain_affine_hull`, `isl_basic_map_intersect`, `isl_basic_map_plain_is_equal` (+2 more); contains control flow with 4 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_basic_map_copy`, `isl_basic_map_plain_affine_hull`, `isl_basic_map_intersect`, `isl_basic_map_plain_is_equal` (+2 more) 相关的例程; 包含控制流结构：4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 3551-3577

````c
	change = coalesce_after_aligning_divs(bmap_i, -1, j, info);

	isl_basic_map_free(bmap_i);

	return change;
}

/* Check if the union of the basic maps represented by info[i] and info[j]
 * can be represented by a single basic map, by aligning or equating
 * their integer divisions.
 * If so, replace the pair by the single basic map and return
 * isl_change_drop_first, isl_change_drop_second or isl_change_fuse.
 * Otherwise, return isl_change_none.
 *
 * Note that we only perform any test if the number of divs is different
 * in the two basic maps.  In case the number of divs is the same,
 * we have already established that the divs are different
 * in the two basic maps.
 * In particular, if the number of divs of basic map i is smaller than
 * the number of divs of basic map j, then we check if j is a subset of i
 * and vice versa.
 */
static enum isl_change coalesce_divs(int i, int j,
	struct isl_coalesce_info *info)
{
	enum isl_change change = isl_change_none;

````
- **EN**: This block declares or references types such as `isl_coalesce_info`; defines enum values such as `isl_change`; declares or defines routines around `coalesce_after_aligning_divs`, `isl_basic_map_free`, `coalesce_divs`; emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_coalesce_info`; 定义枚举类型，例如 `isl_change`; 声明或定义与 `coalesce_after_aligning_divs`, `isl_basic_map_free`, `coalesce_divs` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 3578-3605

````c
	if (info[i].bmap->n_div < info[j].bmap->n_div)
		change = coalesce_after_aligning_divs(info[i].bmap, i, j, info);
	if (change != isl_change_none)
		return change;

	if (info[j].bmap->n_div < info[i].bmap->n_div)
		change = coalesce_after_aligning_divs(info[j].bmap, j, i, info);
	if (change != isl_change_none)
		return invert_change(change);

	change = coalesce_subset_with_equalities(i, j, info);
	if (change != isl_change_none)
		return change;

	change = coalesce_subset_with_equalities(j, i, info);
	if (change != isl_change_none)
		return invert_change(change);

	return isl_change_none;
}

/* Does "bmap" involve any divs that themselves refer to divs?
 */
static isl_bool has_nested_div(__isl_keep isl_basic_map *bmap)
{
	isl_size total;
	isl_size n_div;

````
- **EN**: This block declares or defines routines around `coalesce_after_aligning_divs`, `coalesce_subset_with_equalities`, `has_nested_div`; contains control flow with 6 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `coalesce_after_aligning_divs`, `coalesce_subset_with_equalities`, `has_nested_div` 相关的例程; 包含控制流结构：6 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 3606-3639

````c
	total = isl_basic_map_dim(bmap, isl_dim_all);
	n_div = isl_basic_map_dim(bmap, isl_dim_div);
	if (total < 0 || n_div < 0)
		return isl_bool_error;
	total -= n_div;

	return isl_basic_map_any_div_involves_vars(bmap, total, n_div);
}

/* Return a list of affine expressions, one for each integer division
 * in "bmap_i".  For each integer division that also appears in "bmap_j",
 * the affine expression is set to NaN.  The number of NaNs in the list
 * is equal to the number of integer divisions in "bmap_j".
 * For the other integer divisions of "bmap_i", the corresponding
 * element in the list is a purely affine expression equal to the integer
 * division in "hull".
 * If no such list can be constructed, then the number of elements
 * in the returned list is smaller than the number of integer divisions
 * in "bmap_i".
 * The integer division of "bmap_i" and "bmap_j" are assumed to be known and
 * not contain any nested divs.
 */
static __isl_give isl_aff_list *set_up_substitutions(
	__isl_keep isl_basic_map *bmap_i, __isl_keep isl_basic_map *bmap_j,
	__isl_take isl_basic_map *hull)
{
	isl_size n_div_i, n_div_j, total;
	isl_ctx *ctx;
	isl_local_space *ls;
	isl_basic_set *wrap_hull;
	isl_aff *aff_nan;
	isl_aff_list *list;
	int i, j;

````
- **EN**: This block declares or defines routines around `isl_basic_map_dim`, `set_up_substitutions`; contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_basic_map_dim`, `set_up_substitutions` 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 3640-3670

````c
	n_div_i = isl_basic_map_dim(bmap_i, isl_dim_div);
	n_div_j = isl_basic_map_dim(bmap_j, isl_dim_div);
	total = isl_basic_map_dim(bmap_i, isl_dim_all);
	if (!hull || n_div_i < 0 || n_div_j < 0 || total < 0)
		return NULL;

	ctx = isl_basic_map_get_ctx(hull);
	total -= n_div_i;

	ls = isl_basic_map_get_local_space(bmap_i);
	ls = isl_local_space_wrap(ls);
	wrap_hull = isl_basic_map_wrap(hull);

	aff_nan = isl_aff_nan_on_domain(isl_local_space_copy(ls));
	list = isl_aff_list_alloc(ctx, n_div_i);

	j = 0;
	for (i = 0; i < n_div_i; ++i) {
		isl_aff *aff;
		isl_size n_div;

		if (j < n_div_j &&
		    isl_basic_map_equal_div_expr_part(bmap_i, i, bmap_j, j,
						    0, 2 + total)) {
			++j;
			list = isl_aff_list_add(list, isl_aff_copy(aff_nan));
			continue;
		}
		if (n_div_i - i <= n_div_j - j)
			break;

````
- **EN**: This block declares or defines routines around `isl_basic_map_dim`, `isl_basic_map_get_ctx`, `isl_basic_map_get_local_space`, `isl_local_space_wrap` (+5 more); contains control flow with 1 loop construct(s), 3 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_basic_map_dim`, `isl_basic_map_get_ctx`, `isl_basic_map_get_local_space`, `isl_local_space_wrap` (+5 more) 相关的例程; 包含控制流结构：1 处循环、3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 3671-3698

````c
		aff = isl_local_space_get_div(ls, i);
		aff = isl_aff_substitute_equalities(aff,
						isl_basic_set_copy(wrap_hull));
		aff = isl_aff_floor(aff);
		n_div = isl_aff_dim(aff, isl_dim_div);
		if (n_div < 0)
			goto error;
		if (n_div != 0) {
			isl_aff_free(aff);
			break;
		}

		list = isl_aff_list_add(list, aff);
	}

	isl_aff_free(aff_nan);
	isl_local_space_free(ls);
	isl_basic_set_free(wrap_hull);

	return list;
error:
	isl_aff_free(aff_nan);
	isl_local_space_free(ls);
	isl_basic_set_free(wrap_hull);
	isl_aff_list_free(list);
	return NULL;
}

````
- **EN**: This block declares or defines routines around `isl_local_space_get_div`, `isl_aff_substitute_equalities`, `isl_basic_set_copy`, `isl_aff_floor` (+6 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_local_space_get_div`, `isl_aff_substitute_equalities`, `isl_basic_set_copy`, `isl_aff_floor` (+6 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 3699-3729

````c
/* Add variables to info->bmap and info->tab corresponding to the elements
 * in "list" that are not set to NaN.
 * "extra_var" is the number of these elements.
 * "dim" is the offset in the variables of "tab" where we should
 * start considering the elements in "list".
 * When this function returns, the total number of variables in "tab"
 * is equal to "dim" plus the number of elements in "list".
 *
 * The newly added existentially quantified variables are not given
 * an explicit representation because the corresponding div constraints
 * do not appear in info->bmap.  These constraints are not added
 * to info->bmap because for internal consistency, they would need to
 * be added to info->tab as well, where they could combine with the equality
 * that is added later to result in constraints that do not hold
 * in the original input.
 */
static isl_stat add_sub_vars(struct isl_coalesce_info *info,
	__isl_keep isl_aff_list *list, int dim, int extra_var)
{
	int i, j, d;
	isl_size n;

	info->bmap = isl_basic_map_cow(info->bmap);
	info->bmap = isl_basic_map_extend(info->bmap, extra_var, 0, 0);
	n = isl_aff_list_n_aff(list);
	if (!info->bmap || n < 0)
		return isl_stat_error;
	for (i = 0; i < n; ++i) {
		int is_nan;
		isl_aff *aff;

````
- **EN**: This block declares or defines routines around `add_sub_vars`, `isl_basic_map_cow`, `isl_basic_map_extend`, `isl_aff_list_n_aff`; contains control flow with 1 loop construct(s), 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `add_sub_vars`, `isl_basic_map_cow`, `isl_basic_map_extend`, `isl_aff_list_n_aff` 相关的例程; 包含控制流结构：1 处循环、1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 3730-3761

````c
		aff = isl_aff_list_get_aff(list, i);
		is_nan = isl_aff_is_nan(aff);
		isl_aff_free(aff);
		if (is_nan < 0)
			return isl_stat_error;
		if (is_nan)
			continue;

		if (isl_tab_insert_var(info->tab, dim + i) < 0)
			return isl_stat_error;
		d = isl_basic_map_alloc_div(info->bmap);
		if (d < 0)
			return isl_stat_error;
		info->bmap = isl_basic_map_mark_div_unknown(info->bmap, d);
		for (j = d; j > i; --j)
			info->bmap = isl_basic_map_swap_div(info->bmap,
							    j - 1, j);
		if (!info->bmap)
			return isl_stat_error;
	}

	return isl_stat_ok;
}

/* For each element in "list" that is not set to NaN, fix the corresponding
 * variable in "tab" to the purely affine expression defined by the element.
 * "dim" is the offset in the variables of "tab" where we should
 * start considering the elements in "list".
 *
 * This function assumes that a sufficient number of rows and
 * elements in the constraint array are available in the tableau.
 */
````
- **EN**: This block declares or defines routines around `isl_aff_list_get_aff`, `isl_aff_is_nan`, `isl_aff_free`, `isl_basic_map_alloc_div` (+2 more); contains control flow with 1 loop construct(s), 5 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_aff_list_get_aff`, `isl_aff_is_nan`, `isl_aff_free`, `isl_basic_map_alloc_div` (+2 more) 相关的例程; 包含控制流结构：1 处循环、5 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 3762-3796

````c
static isl_stat add_sub_equalities(struct isl_tab *tab,
	__isl_keep isl_aff_list *list, int dim)
{
	int i;
	isl_size n;
	isl_ctx *ctx;
	isl_vec *sub;
	isl_aff *aff;

	n = isl_aff_list_n_aff(list);
	if (n < 0)
		return isl_stat_error;

	ctx = isl_tab_get_ctx(tab);
	sub = isl_vec_alloc(ctx, 1 + dim + n);
	if (!sub)
		return isl_stat_error;
	isl_seq_clr(sub->el + 1 + dim, n);

	for (i = 0; i < n; ++i) {
		aff = isl_aff_list_get_aff(list, i);
		if (!aff)
			goto error;
		if (isl_aff_is_nan(aff)) {
			isl_aff_free(aff);
			continue;
		}
		isl_seq_cpy(sub->el, aff->v->el + 1, 1 + dim);
		isl_int_neg(sub->el[1 + dim + i], aff->v->el[0]);
		if (isl_tab_add_eq(tab, sub->el) < 0)
			goto error;
		isl_int_set_si(sub->el[1 + dim + i], 0);
		isl_aff_free(aff);
	}

````
- **EN**: This block declares or defines routines around `add_sub_equalities`, `isl_aff_list_n_aff`, `isl_tab_get_ctx`, `isl_vec_alloc` (+6 more); contains control flow with 1 loop construct(s), 5 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `add_sub_equalities`, `isl_aff_list_n_aff`, `isl_tab_get_ctx`, `isl_vec_alloc` (+6 more) 相关的例程; 包含控制流结构：1 处循环、5 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 3797-3822

````c
	isl_vec_free(sub);
	return isl_stat_ok;
error:
	isl_aff_free(aff);
	isl_vec_free(sub);
	return isl_stat_error;
}

/* Add variables to info->tab and info->bmap corresponding to the elements
 * in "list" that are not set to NaN.  The value of the added variable
 * in info->tab is fixed to the purely affine expression defined by the element.
 * "dim" is the offset in the variables of info->tab where we should
 * start considering the elements in "list".
 * When this function returns, the total number of variables in info->tab
 * is equal to "dim" plus the number of elements in "list".
 */
static isl_stat add_subs(struct isl_coalesce_info *info,
	__isl_keep isl_aff_list *list, int dim)
{
	int extra_var;
	isl_size n;

	n = isl_aff_list_n_aff(list);
	if (n < 0)
		return isl_stat_error;

````
- **EN**: This block declares or defines routines around `isl_vec_free`, `isl_aff_free`, `add_subs`, `isl_aff_list_n_aff`; contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_vec_free`, `isl_aff_free`, `add_subs`, `isl_aff_list_n_aff` 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 3823-3855

````c
	extra_var = n - (info->tab->n_var - dim);

	if (isl_tab_extend_vars(info->tab, extra_var) < 0)
		return isl_stat_error;
	if (isl_tab_extend_cons(info->tab, 2 * extra_var) < 0)
		return isl_stat_error;
	if (add_sub_vars(info, list, dim, extra_var) < 0)
		return isl_stat_error;

	return add_sub_equalities(info->tab, list, dim);
}

/* Coalesce basic map "j" into basic map "i" after adding the extra integer
 * divisions in "i" but not in "j" to basic map "j", with values
 * specified by "list".  The total number of elements in "list"
 * is equal to the number of integer divisions in "i", while the number
 * of NaN elements in the list is equal to the number of integer divisions
 * in "j".
 *
 * If no coalescing can be performed, then we need to revert basic map "j"
 * to its original state.  We do the same if basic map "i" gets dropped
 * during the coalescing, even though this should not happen in practice
 * since we have already checked for "j" being a subset of "i"
 * before we reach this stage.
 */
static enum isl_change coalesce_with_subs(int i, int j,
	struct isl_coalesce_info *info, __isl_keep isl_aff_list *list)
{
	isl_basic_map *bmap_j;
	struct isl_tab_undo *snap;
	isl_size dim, n_div;
	enum isl_change change;

````
- **EN**: This block declares or references types such as `isl_coalesce_info`, `isl_tab_undo`; defines enum values such as `isl_change`; declares or defines routines around `coalesce_with_subs`; contains control flow with 3 conditional check(s); and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_coalesce_info`, `isl_tab_undo`; 定义枚举类型，例如 `isl_change`; 声明或定义与 `coalesce_with_subs` 相关的例程; 包含控制流结构：3 处条件判断；并延续周边实现细节。

### Lines 3856-3883

````c
	bmap_j = isl_basic_map_copy(info[j].bmap);
	snap = isl_tab_snap(info[j].tab);

	dim = isl_basic_map_dim(bmap_j, isl_dim_all);
	n_div = isl_basic_map_dim(bmap_j, isl_dim_div);
	if (dim < 0 || n_div < 0)
		goto error;
	dim -= n_div;
	if (add_subs(&info[j], list, dim) < 0)
		goto error;

	change = coalesce_local_pair(i, j, info);
	if (change != isl_change_none && change != isl_change_drop_first) {
		isl_basic_map_free(bmap_j);
	} else {
		isl_basic_map_free(info[j].bmap);
		info[j].bmap = bmap_j;

		if (isl_tab_rollback(info[j].tab, snap) < 0)
			return isl_change_error;
	}

	return change;
error:
	isl_basic_map_free(bmap_j);
	return isl_change_error;
}

````
- **EN**: This block declares or defines routines around `isl_basic_map_copy`, `isl_tab_snap`, `isl_basic_map_dim`, `coalesce_local_pair` (+1 more); contains control flow with 4 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_basic_map_copy`, `isl_tab_snap`, `isl_basic_map_dim`, `coalesce_local_pair` (+1 more) 相关的例程; 包含控制流结构：4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 3884-3915

````c
/* Check if we can coalesce basic map "j" into basic map "i" after copying
 * those extra integer divisions in "i" that can be simplified away
 * using the extra equalities in "j".
 * All divs are assumed to be known and not contain any nested divs.
 *
 * We first check if there are any extra equalities in "j" that we
 * can exploit.  Then we check if every integer division in "i"
 * either already appears in "j" or can be simplified using the
 * extra equalities to a purely affine expression.
 * If these tests succeed, then we try to coalesce the two basic maps
 * by introducing extra dimensions in "j" corresponding to
 * the extra integer divisions "i" fixed to the corresponding
 * purely affine expression.
 */
static enum isl_change check_coalesce_into_eq(int i, int j,
	struct isl_coalesce_info *info)
{
	isl_size n_div_i, n_div_j, n;
	isl_basic_map *hull_i, *hull_j;
	isl_bool equal, empty;
	isl_aff_list *list;
	enum isl_change change;

	n_div_i = isl_basic_map_dim(info[i].bmap, isl_dim_div);
	n_div_j = isl_basic_map_dim(info[j].bmap, isl_dim_div);
	if (n_div_i < 0 || n_div_j < 0)
		return isl_change_error;
	if (n_div_i <= n_div_j)
		return isl_change_none;
	if (info[j].bmap->n_eq == 0)
		return isl_change_none;

````
- **EN**: This block declares or references types such as `isl_coalesce_info`; defines enum values such as `isl_change`; declares or defines routines around `check_coalesce_into_eq`, `isl_basic_map_dim`; contains control flow with 3 conditional check(s); and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_coalesce_info`; 定义枚举类型，例如 `isl_change`; 声明或定义与 `check_coalesce_into_eq`, `isl_basic_map_dim` 相关的例程; 包含控制流结构：3 处条件判断；并延续周边实现细节。

### Lines 3916-3943

````c
	hull_i = isl_basic_map_copy(info[i].bmap);
	hull_i = isl_basic_map_plain_affine_hull(hull_i);
	hull_j = isl_basic_map_copy(info[j].bmap);
	hull_j = isl_basic_map_plain_affine_hull(hull_j);

	hull_j = isl_basic_map_intersect(hull_j, isl_basic_map_copy(hull_i));
	equal = isl_basic_map_plain_is_equal(hull_i, hull_j);
	empty = isl_basic_map_plain_is_empty(hull_j);
	isl_basic_map_free(hull_i);

	if (equal < 0 || empty < 0)
		goto error;
	if (equal || empty) {
		isl_basic_map_free(hull_j);
		return isl_change_none;
	}

	list = set_up_substitutions(info[i].bmap, info[j].bmap, hull_j);
	if (!list)
		return isl_change_error;
	n = isl_aff_list_n_aff(list);
	if (n < 0)
		change = isl_change_error;
	else if (n < n_div_i)
		change = isl_change_none;
	else
		change = coalesce_with_subs(i, j, info, list);

````
- **EN**: This block declares or defines routines around `isl_basic_map_copy`, `isl_basic_map_plain_affine_hull`, `isl_basic_map_intersect`, `isl_basic_map_plain_is_equal` (+5 more); contains control flow with 5 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_basic_map_copy`, `isl_basic_map_plain_affine_hull`, `isl_basic_map_intersect`, `isl_basic_map_plain_is_equal` (+5 more) 相关的例程; 包含控制流结构：5 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 3944-3977

````c
	isl_aff_list_free(list);

	return change;
error:
	isl_basic_map_free(hull_j);
	return isl_change_error;
}

/* Check if we can coalesce basic maps "i" and "j" after copying
 * those extra integer divisions in one of the basic maps that can
 * be simplified away using the extra equalities in the other basic map.
 * We require all divs to be known in both basic maps.
 * Furthermore, to simplify the comparison of div expressions,
 * we do not allow any nested integer divisions.
 */
static enum isl_change check_coalesce_eq(int i, int j,
	struct isl_coalesce_info *info)
{
	isl_bool known, nested;
	enum isl_change change;

	known = isl_basic_map_divs_known(info[i].bmap);
	if (known < 0 || !known)
		return known < 0 ? isl_change_error : isl_change_none;
	known = isl_basic_map_divs_known(info[j].bmap);
	if (known < 0 || !known)
		return known < 0 ? isl_change_error : isl_change_none;
	nested = has_nested_div(info[i].bmap);
	if (nested < 0 || nested)
		return nested < 0 ? isl_change_error : isl_change_none;
	nested = has_nested_div(info[j].bmap);
	if (nested < 0 || nested)
		return nested < 0 ? isl_change_error : isl_change_none;

````
- **EN**: This block declares or references types such as `isl_coalesce_info`; defines enum values such as `isl_change`; declares or defines routines around `isl_aff_list_free`, `isl_basic_map_free`, `check_coalesce_eq`, `isl_basic_map_divs_known` (+1 more); contains control flow with 4 conditional check(s); and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_coalesce_info`; 定义枚举类型，例如 `isl_change`; 声明或定义与 `isl_aff_list_free`, `isl_basic_map_free`, `check_coalesce_eq`, `isl_basic_map_divs_known` (+1 more) 相关的例程; 包含控制流结构：4 处条件判断；并延续周边实现细节。

### Lines 3978-4015

````c
	change = check_coalesce_into_eq(i, j, info);
	if (change != isl_change_none)
		return change;
	change = check_coalesce_into_eq(j, i, info);
	if (change != isl_change_none)
		return invert_change(change);

	return isl_change_none;
}

/* Check if the union of the given pair of basic maps
 * can be represented by a single basic map.
 * If so, replace the pair by the single basic map and return
 * isl_change_drop_first, isl_change_drop_second or isl_change_fuse.
 * Otherwise, return isl_change_none.
 *
 * We first check if the two basic maps live in the same local space,
 * after aligning the divs that differ by only an integer constant.
 * If so, we do the complete check.  Otherwise, we check if they have
 * the same number of integer divisions and can be coalesced, if one is
 * an obvious subset of the other or if the extra integer divisions
 * of one basic map can be simplified away using the extra equalities
 * of the other basic map.
 *
 * Note that trying to coalesce pairs of disjuncts with the same
 * number, but different local variables may drop the explicit
 * representation of some of these local variables.
 * This operation is therefore not performed when
 * the "coalesce_preserve_locals" option is set.
 */
static enum isl_change coalesce_pair(int i, int j,
	struct isl_coalesce_info *info)
{
	int preserve;
	isl_bool same;
	enum isl_change change;
	isl_ctx *ctx;

````
- **EN**: This block declares or references types such as `isl_coalesce_info`; defines enum values such as `isl_change`; declares or defines routines around `check_coalesce_into_eq`, `coalesce_pair`; contains control flow with 2 conditional check(s); and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_coalesce_info`; 定义枚举类型，例如 `isl_change`; 声明或定义与 `check_coalesce_into_eq`, `coalesce_pair` 相关的例程; 包含控制流结构：2 处条件判断；并延续周边实现细节。

### Lines 4016-4045

````c
	if (harmonize_divs(&info[i], &info[j]) < 0)
		return isl_change_error;
	same = same_divs(info[i].bmap, info[j].bmap);
	if (same < 0)
		return isl_change_error;
	if (same)
		return coalesce_local_pair(i, j, info);

	ctx = isl_basic_map_get_ctx(info[i].bmap);
	preserve = isl_options_get_coalesce_preserve_locals(ctx);
	if (!preserve && info[i].bmap->n_div == info[j].bmap->n_div) {
		change = coalesce_local_pair(i, j, info);
		if (change != isl_change_none)
			return change;
	}

	change = coalesce_divs(i, j, info);
	if (change != isl_change_none)
		return change;

	return check_coalesce_eq(i, j, info);
}

/* Return the maximum of "a" and "b".
 */
static int isl_max(int a, int b)
{
	return a > b ? a : b;
}

````
- **EN**: This block declares or defines routines around `same_divs`, `isl_basic_map_get_ctx`, `isl_options_get_coalesce_preserve_locals`, `coalesce_local_pair` (+2 more); contains control flow with 6 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `same_divs`, `isl_basic_map_get_ctx`, `isl_options_get_coalesce_preserve_locals`, `coalesce_local_pair` (+2 more) 相关的例程; 包含控制流结构：6 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 4046-4077

````c
/* Pairwise coalesce the basic maps in the range [start1, end1[ of "info"
 * with those in the range [start2, end2[, skipping basic maps
 * that have been removed (either before or within this function).
 *
 * For each basic map i in the first range, we check if it can be coalesced
 * with respect to any previously considered basic map j in the second range.
 * If i gets dropped (because it was a subset of some j), then
 * we can move on to the next basic map.
 * If j gets dropped, we need to continue checking against the other
 * previously considered basic maps.
 * If the two basic maps got fused, then we recheck the fused basic map
 * against the previously considered basic maps, starting at i + 1
 * (even if start2 is greater than i + 1).
 */
static int coalesce_range(isl_ctx *ctx, struct isl_coalesce_info *info,
	int start1, int end1, int start2, int end2)
{
	int i, j;

	for (i = end1 - 1; i >= start1; --i) {
		if (info[i].removed)
			continue;
		for (j = isl_max(i + 1, start2); j < end2; ++j) {
			enum isl_change changed;

			if (info[j].removed)
				continue;
			if (info[i].removed)
				isl_die(ctx, isl_error_internal,
					"basic map unexpectedly removed",
					return -1);
			changed = coalesce_pair(i, j, info);
````
- **EN**: This block defines enum values such as `isl_change`; declares or defines routines around `removed`, `dropped`, `coalesce_range`, `isl_die` (+1 more); contains control flow with 2 loop construct(s), 3 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 定义枚举类型，例如 `isl_change`; 声明或定义与 `removed`, `dropped`, `coalesce_range`, `isl_die` (+1 more) 相关的例程; 包含控制流结构：2 处循环、3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 4078-4108

````c
			switch (changed) {
			case isl_change_error:
				return -1;
			case isl_change_none:
			case isl_change_drop_second:
				continue;
			case isl_change_drop_first:
				j = end2;
				break;
			case isl_change_fuse:
				j = i;
				break;
			}
		}
	}

	return 0;
}

/* Pairwise coalesce the basic maps described by the "n" elements of "info".
 *
 * We consider groups of basic maps that live in the same apparent
 * affine hull and we first coalesce within such a group before we
 * coalesce the elements in the group with elements of previously
 * considered groups.  If a fuse happens during the second phase,
 * then we also reconsider the elements within the group.
 */
static int coalesce(isl_ctx *ctx, int n, struct isl_coalesce_info *info)
{
	int start, end;

````
- **EN**: This block declares or defines routines around `coalesce`; contains control flow with 1 switch dispatch(es); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `coalesce` 相关的例程; 包含控制流结构：1 处分支派发; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 4109-4139

````c
	for (end = n; end > 0; end = start) {
		start = end - 1;
		while (start >= 1 &&
		    info[start - 1].hull_hash == info[start].hull_hash)
			start--;
		if (coalesce_range(ctx, info, start, end, start, end) < 0)
			return -1;
		if (coalesce_range(ctx, info, start, end, end, n) < 0)
			return -1;
	}

	return 0;
}

/* Update the basic maps in "map" based on the information in "info".
 * In particular, remove the basic maps that have been marked removed and
 * update the others based on the information in the corresponding tableau.
 * Since we detected implicit equalities without calling
 * isl_basic_map_gauss, we need to do it now.
 * Also call isl_basic_map_simplify if we may have lost the definition
 * of one or more integer divisions.
 * If a basic map is still equal to the one from which the corresponding "info"
 * entry was created, then redundant constraint and
 * implicit equality constraint detection have been performed
 * on the corresponding tableau and the basic map can be marked as such.
 */
static __isl_give isl_map *update_basic_maps(__isl_take isl_map *map,
	int n, struct isl_coalesce_info *info)
{
	int i;

````
- **EN**: This block declares or defines routines around `update_basic_maps`; contains control flow with 2 loop construct(s), 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `update_basic_maps` 相关的例程; 包含控制流结构：2 处循环、2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 4140-4168

````c
	if (!map)
		return NULL;

	for (i = n - 1; i >= 0; --i) {
		if (info[i].removed) {
			isl_basic_map_free(map->p[i]);
			if (i != map->n - 1)
				map->p[i] = map->p[map->n - 1];
			map->n--;
			continue;
		}

		info[i].bmap = isl_basic_map_update_from_tab(info[i].bmap,
							info[i].tab);
		info[i].bmap = isl_basic_map_gauss(info[i].bmap, NULL);
		if (info[i].simplify)
			info[i].bmap = isl_basic_map_simplify(info[i].bmap);
		info[i].bmap = isl_basic_map_finalize(info[i].bmap);
		if (!info[i].bmap)
			return isl_map_free(map);
		if (!info[i].modified) {
			ISL_F_SET(info[i].bmap, ISL_BASIC_MAP_NO_IMPLICIT);
			ISL_F_SET(info[i].bmap, ISL_BASIC_MAP_NO_REDUNDANT);
		}
		isl_basic_map_free(map->p[i]);
		map->p[i] = info[i].bmap;
		info[i].bmap = NULL;
	}

````
- **EN**: This block declares or defines routines around `isl_basic_map_free`, `isl_basic_map_update_from_tab`, `isl_basic_map_gauss`, `isl_basic_map_simplify` (+2 more); contains control flow with 1 loop construct(s), 6 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_basic_map_free`, `isl_basic_map_update_from_tab`, `isl_basic_map_gauss`, `isl_basic_map_simplify` (+2 more) 相关的例程; 包含控制流结构：1 处循环、6 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 4169-4203

````c
	return map;
}

/* For each pair of basic maps in the map, check if the union of the two
 * can be represented by a single basic map.
 * If so, replace the pair by the single basic map and start over.
 *
 * We factor out any (hidden) common factor from the constraint
 * coefficients to improve the detection of adjacent constraints.
 * Note that this function does not call isl_basic_map_gauss,
 * but it does make sure that only a single copy of the basic map
 * is affected.  This means that isl_basic_map_gauss may have
 * to be called at the end of the computation (in update_basic_maps)
 * on this single copy to ensure that
 * the basic maps are not left in an unexpected state.
 *
 * Since we are constructing the tableaus of the basic maps anyway,
 * we exploit them to detect implicit equalities and redundant constraints.
 * This also helps the coalescing as it can ignore the redundant constraints.
 * In order to avoid confusion, we make all implicit equalities explicit
 * in the basic maps.  If the basic map only has a single reference
 * (this happens in particular if it was modified by
 * isl_basic_map_reduce_coefficients), then isl_basic_map_gauss
 * does not get called on the result.  The call to
 * isl_basic_map_gauss in update_basic_maps resolves this as well.
 * For each basic map, we also compute the hash of the apparent affine hull
 * for use in coalesce.
 */
__isl_give isl_map *isl_map_coalesce(__isl_take isl_map *map)
{
	int i;
	unsigned n;
	isl_ctx *ctx;
	struct isl_coalesce_info *info = NULL;

````
- **EN**: This block declares or references types such as `isl_coalesce_info`; declares or defines routines around `any`, `computation`, `isl_map_coalesce`; emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_coalesce_info`; 声明或定义与 `any`, `computation`, `isl_map_coalesce` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 4204-4235

````c
	map = isl_map_remove_empty_parts(map);
	map = isl_map_remove_obvious_duplicates(map);
	if (!map)
		return NULL;

	if (map->n <= 1)
		return map;

	ctx = isl_map_get_ctx(map);
	map = isl_map_sort_divs(map);
	map = isl_map_cow(map);

	if (!map)
		return NULL;

	n = map->n;

	info = isl_calloc_array(map->ctx, struct isl_coalesce_info, n);
	if (!info)
		goto error;

	for (i = 0; i < map->n; ++i) {
		map->p[i] = isl_basic_map_reduce_coefficients(map->p[i]);
		if (!map->p[i])
			goto error;
		info[i].bmap = isl_basic_map_copy(map->p[i]);
		info[i].tab = isl_tab_from_basic_map(info[i].bmap, 0);
		if (!info[i].tab)
			goto error;
		if (!ISL_F_ISSET(info[i].bmap, ISL_BASIC_MAP_NO_IMPLICIT))
			if (isl_tab_detect_implicit_equalities(info[i].tab) < 0)
				goto error;
````
- **EN**: This block declares or defines routines around `isl_map_remove_empty_parts`, `isl_map_remove_obvious_duplicates`, `isl_map_get_ctx`, `isl_map_sort_divs` (+5 more); contains control flow with 1 loop construct(s), 8 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_map_remove_empty_parts`, `isl_map_remove_obvious_duplicates`, `isl_map_get_ctx`, `isl_map_sort_divs` (+5 more) 相关的例程; 包含控制流结构：1 处循环、8 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 4236-4263

````c
		info[i].bmap = isl_tab_make_equalities_explicit(info[i].tab,
								info[i].bmap);
		if (!info[i].bmap)
			goto error;
		if (!ISL_F_ISSET(info[i].bmap, ISL_BASIC_MAP_NO_REDUNDANT))
			if (isl_tab_detect_redundant(info[i].tab) < 0)
				goto error;
		if (coalesce_info_set_hull_hash(&info[i]) < 0)
			goto error;
	}
	for (i = map->n - 1; i >= 0; --i)
		if (info[i].tab->empty)
			drop(&info[i]);

	if (coalesce(ctx, n, info) < 0)
		goto error;

	map = update_basic_maps(map, n, info);

	clear_coalesce_info(n, info);

	return map;
error:
	clear_coalesce_info(n, info);
	isl_map_free(map);
	return NULL;
}

````
- **EN**: This block declares or defines routines around `isl_tab_make_equalities_explicit`, `drop`, `update_basic_maps`, `clear_coalesce_info` (+1 more); contains control flow with 1 loop construct(s), 6 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_tab_make_equalities_explicit`, `drop`, `update_basic_maps`, `clear_coalesce_info` (+1 more) 相关的例程; 包含控制流结构：1 处循环、6 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 4264-4271

````c
/* For each pair of basic sets in the set, check if the union of the two
 * can be represented by a single basic set.
 * If so, replace the pair by the single basic set and start over.
 */
__isl_give isl_set *isl_set_coalesce(__isl_take isl_set *set)
{
	return set_from_map(isl_map_coalesce(set_to_map(set)));
}
````
- **EN**: This block declares or defines routines around `isl_set_coalesce`; emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `isl_set_coalesce` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理; 保留用于解释意图、用法或算法背景的注释.

## Key Concepts / 关键概念

- **ISL relations and sets**
  - **CN**: ISL 关系与集合
- **Template/type wrappers**
  - **CN**: 模板与类型包装
- **Loop transformation**
  - **CN**: 循环变换
- **Memory access tracking**
  - **CN**: 内存访问跟踪
- **Schedule construction**
  - **CN**: 调度构建

## Dependencies / 依赖关系

- **ISL headers**: `isl/options.h` — ISL relation/set primitives that support polyhedral analysis and transformations.
  **ISL headers（CN）**：`isl/options.h` —— 支撑多面体分析与变换的 ISL 关系/集合原语。
- **System/standard headers**: `isl_ctx_private.h`, `isl_map_private.h`, `isl_seq.h`, `isl_tab.h`, `isl_mat_private.h`, `isl_local_space_private.h`, `isl_val_private.h`, `isl_vec_private.h` (+5 more) — Standard-library or system declarations required by the implementation.
  **System/standard headers（CN）**：`isl_ctx_private.h`, `isl_map_private.h`, `isl_seq.h`, `isl_tab.h`, `isl_mat_private.h`, `isl_local_space_private.h`, `isl_val_private.h`, `isl_vec_private.h` (+5 more) —— 实现所需的标准库或系统声明。
