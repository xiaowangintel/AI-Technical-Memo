# isl_fold.c — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `polly/lib/External/isl/isl_fold.c` | `polly/lib/External/isl/isl_fold.c` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | Implements or embeds upstream ISL support code used by Polly. | 实现或内嵌 Polly 使用的上游 ISL 支撑代码。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-27

````c
/*
 * Copyright 2010      INRIA Saclay
 *
 * Use of this software is governed by the MIT license
 *
 * Written by Sven Verdoolaege, INRIA Saclay - Ile-de-France,
 * Parc Club Orsay Universite, ZAC des vignes, 4 rue Jacques Monod,
 * 91893 Orsay, France 
 */

#include <isl_map_private.h>
#include <isl_union_map_private.h>
#include <isl_polynomial_private.h>
#include <isl_point_private.h>
#include <isl_space_private.h>
#include <isl_lp_private.h>
#include <isl_seq.h>
#include <isl_mat_private.h>
#include <isl_val_private.h>
#include <isl_vec_private.h>
#include <isl_config.h>

#undef EL_BASE
#define EL_BASE pw_qpolynomial_fold

#include <isl_list_templ.c>

````
- **EN**: This block imports system/standard headers needed by the surrounding code; uses preprocessor directives to control compilation, macros, or include guards; defines macros like `EL_BASE`; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 引入周边逻辑所需的 system/standard 头文件; 使用预处理指令控制编译、宏或头文件保护; 定义宏，例如 `EL_BASE`; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 28-53

````c
enum isl_fold isl_fold_type_negate(enum isl_fold type)
{
	switch (type) {
	case isl_fold_error:
		return isl_fold_error;
	case isl_fold_min:
		return isl_fold_max;
	case isl_fold_max:
		return isl_fold_min;
	case isl_fold_list:
		return isl_fold_list;
	}

	isl_die(NULL, isl_error_internal, "unhandled isl_fold type", abort());
}

/* Construct a new reduction with the given type, domain space and
 * list of polynomials.
 */
static __isl_give isl_qpolynomial_fold *qpolynomial_fold_alloc(
	enum isl_fold type, __isl_take isl_space *space,
	__isl_take isl_qpolynomial_list *list)
{
	isl_ctx *ctx;
	isl_qpolynomial_fold *fold;

````
- **EN**: This block defines enum values such as `isl_fold`; declares or defines routines around `isl_fold_type_negate`, `isl_die`, `qpolynomial_fold_alloc`; contains control flow with 1 switch dispatch(es); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 定义枚举类型，例如 `isl_fold`; 声明或定义与 `isl_fold_type_negate`, `isl_die`, `qpolynomial_fold_alloc` 相关的例程; 包含控制流结构：1 处分支派发; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 54-86

````c
	if (type < 0 || !space || !list)
		goto error;

	ctx = isl_space_get_ctx(space);
	fold = isl_calloc_type(ctx, struct isl_qpolynomial_fold);
	if (!fold)
		goto error;

	fold->ref = 1;
	fold->type = type;
	fold->dim = space;
	fold->list = list;

	return fold;
error:
	isl_space_free(space);
	isl_qpolynomial_list_free(list);
	return NULL;
}

isl_ctx *isl_qpolynomial_fold_get_ctx(__isl_keep isl_qpolynomial_fold *fold)
{
	return fold ? fold->dim->ctx : NULL;
}

/* Return the domain space of "fold".
 */
static __isl_keep isl_space *isl_qpolynomial_fold_peek_domain_space(
	__isl_keep isl_qpolynomial_fold *fold)
{
	return fold ? fold->dim : NULL;
}

````
- **EN**: This block declares or defines routines around `isl_space_get_ctx`, `isl_calloc_type`, `isl_space_free`, `isl_qpolynomial_list_free` (+2 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_space_get_ctx`, `isl_calloc_type`, `isl_space_free`, `isl_qpolynomial_list_free` (+2 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 87-115

````c
__isl_give isl_space *isl_qpolynomial_fold_get_domain_space(
	__isl_keep isl_qpolynomial_fold *fold)
{
	return isl_space_copy(isl_qpolynomial_fold_peek_domain_space(fold));
}

/* Return the space of the domain of "fold".
 * This may be either a copy or the space itself
 * if there is only one reference to "fold".
 * This allows the space to be modified inplace
 * if both the expression and its space have only a single reference.
 * The caller is not allowed to modify "fold" between this call and
 * a subsequent call to isl_qpolynomial_fold_restore_domain_space.
 * The only exception is that isl_qpolynomial_fold_free can be called instead.
 */
static __isl_give isl_space *isl_qpolynomial_fold_take_domain_space(
	__isl_keep isl_qpolynomial_fold *fold)
{
	isl_space *space;

	if (!fold)
		return NULL;
	if (fold->ref != 1)
		return isl_qpolynomial_fold_get_domain_space(fold);
	space = fold->dim;
	fold->dim = NULL;
	return space;
}

````
- **EN**: This block declares or defines routines around `isl_qpolynomial_fold_get_domain_space`, `isl_qpolynomial_fold_take_domain_space`; contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_qpolynomial_fold_get_domain_space`, `isl_qpolynomial_fold_take_domain_space` 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 116-146

````c
/* Set the space of the domain of "fold" to "space",
 * where the space of "fold" may be missing
 * due to a preceding call to isl_qpolynomial_fold_take_domain_space.
 * However, in this case, "fold" only has a single reference and
 * then the call to isl_qpolynomial_fold_cow has no effect.
 */
static
__isl_give isl_qpolynomial_fold *isl_qpolynomial_fold_restore_domain_space(
	__isl_keep isl_qpolynomial_fold *fold, __isl_take isl_space *space)
{
	if (!fold || !space)
		goto error;

	if (fold->dim == space) {
		isl_space_free(space);
		return fold;
	}

	fold = isl_qpolynomial_fold_cow(fold);
	if (!fold)
		goto error;
	isl_space_free(fold->dim);
	fold->dim = space;

	return fold;
error:
	isl_qpolynomial_fold_free(fold);
	isl_space_free(space);
	return NULL;
}

````
- **EN**: This block declares or defines routines around `isl_qpolynomial_fold_restore_domain_space`, `isl_space_free`, `isl_qpolynomial_fold_cow`, `isl_qpolynomial_fold_free`; contains control flow with 3 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_qpolynomial_fold_restore_domain_space`, `isl_space_free`, `isl_qpolynomial_fold_cow`, `isl_qpolynomial_fold_free` 相关的例程; 包含控制流结构：3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 147-174

````c
__isl_give isl_space *isl_qpolynomial_fold_get_space(
	__isl_keep isl_qpolynomial_fold *fold)
{
	isl_space *space;
	if (!fold)
		return NULL;
	space = isl_space_copy(fold->dim);
	space = isl_space_from_domain(space);
	space = isl_space_add_dims(space, isl_dim_out, 1);
	return space;
}

/* Return the list of polynomials in the reduction "fold".
 */
__isl_keep isl_qpolynomial_list *isl_qpolynomial_fold_peek_list(
	__isl_keep isl_qpolynomial_fold *fold)
{
	return fold ? fold->list : NULL;
}

/* Return a copy of the list of polynomials in the reduction "fold".
 */
static __isl_give isl_qpolynomial_list *isl_qpolynomial_fold_get_list(
	__isl_keep isl_qpolynomial_fold *fold)
{
	return isl_qpolynomial_list_copy(isl_qpolynomial_fold_peek_list(fold));
}

````
- **EN**: This block declares or defines routines around `isl_qpolynomial_fold_get_space`, `isl_space_copy`, `isl_space_from_domain`, `isl_space_add_dims` (+2 more); contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_qpolynomial_fold_get_space`, `isl_space_copy`, `isl_space_from_domain`, `isl_space_add_dims` (+2 more) 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 175-210

````c
/* Return the list of polynomials of "fold".
 * This may be either a copy or the list itself
 * if there is only one reference to "fold".
 * This allows the list to be modified inplace
 * if both the expression and its list have only a single reference.
 * The caller is not allowed to modify "fold" between this call and
 * a subsequent call to isl_qpolynomial_fold_restore_list.
 * The only exception is that isl_qpolynomial_fold_free can be called instead.
 */
static __isl_give isl_qpolynomial_list *isl_qpolynomial_fold_take_list(
	__isl_keep isl_qpolynomial_fold *fold)
{
	isl_qpolynomial_list *list;

	if (!fold)
		return NULL;
	if (fold->ref != 1)
		return isl_qpolynomial_fold_get_list(fold);
	list = fold->list;
	fold->list = NULL;
	return list;
}

/* Set the space of the list of polynomials of "fold" to "space",
 * where the list of polynomials of "fold" may be missing
 * due to a preceding call to isl_qpolynomial_fold_take_list.
 * However, in this case, "fold" only has a single reference and
 * then the call to isl_qpolynomial_fold_cow has no effect.
 */
static __isl_give isl_qpolynomial_fold *isl_qpolynomial_fold_restore_list(
	__isl_keep isl_qpolynomial_fold *fold,
	__isl_take isl_qpolynomial_list *list)
{
	if (!fold || !list)
		goto error;

````
- **EN**: This block declares or defines routines around `isl_qpolynomial_fold_take_list`, `isl_qpolynomial_fold_restore_list`; contains control flow with 3 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_qpolynomial_fold_take_list`, `isl_qpolynomial_fold_restore_list` 相关的例程; 包含控制流结构：3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 211-236

````c
	if (fold->list == list) {
		isl_qpolynomial_list_free(list);
		return fold;
	}

	fold = isl_qpolynomial_fold_cow(fold);
	if (!fold)
		goto error;
	isl_qpolynomial_list_free(fold->list);
	fold->list = list;

	return fold;
error:
	isl_qpolynomial_fold_free(fold);
	isl_qpolynomial_list_free(list);
	return NULL;
}

/* isl_qpolynomial_list_map callback that calls
 * isl_qpolynomial_reset_domain_space on "qp".
 */
static __isl_give isl_qpolynomial *reset_domain_space(
	__isl_take isl_qpolynomial *qp, void *user)
{
	isl_space *space = user;

````
- **EN**: This block declares or defines routines around `isl_qpolynomial_list_free`, `isl_qpolynomial_fold_cow`, `isl_qpolynomial_fold_free`, `reset_domain_space`; contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_qpolynomial_list_free`, `isl_qpolynomial_fold_cow`, `isl_qpolynomial_fold_free`, `reset_domain_space` 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 237-271

````c
	return isl_qpolynomial_reset_domain_space(qp, isl_space_copy(space));
}

/* Replace the domain space of "fold" by "space".
 *
 * Replace the domain space itself and that of all polynomials
 * in the list.
 */
__isl_give isl_qpolynomial_fold *isl_qpolynomial_fold_reset_domain_space(
	__isl_take isl_qpolynomial_fold *fold, __isl_take isl_space *space)
{
	isl_qpolynomial_list *list;

	list = isl_qpolynomial_fold_take_list(fold);
	list = isl_qpolynomial_list_map(list, &reset_domain_space, space);
	fold = isl_qpolynomial_fold_restore_list(fold, list);

	isl_space_free(isl_qpolynomial_fold_take_domain_space(fold));
	fold = isl_qpolynomial_fold_restore_domain_space(fold, space);

	return fold;
}

/* Reset the space of "fold".  This function is called from isl_pw_templ.c
 * and doesn't know if the space of an element object is represented
 * directly or through its domain.  It therefore passes along both.
 */
__isl_give isl_qpolynomial_fold *isl_qpolynomial_fold_reset_space_and_domain(
	__isl_take isl_qpolynomial_fold *fold, __isl_take isl_space *space,
	__isl_take isl_space *domain)
{
	isl_space_free(space);
	return isl_qpolynomial_fold_reset_domain_space(fold, domain);
}

````
- **EN**: This block declares or defines routines around `isl_qpolynomial_fold_reset_domain_space`, `isl_qpolynomial_fold_take_list`, `isl_qpolynomial_list_map`, `isl_qpolynomial_fold_restore_list` (+3 more); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `isl_qpolynomial_fold_reset_domain_space`, `isl_qpolynomial_fold_take_list`, `isl_qpolynomial_list_map`, `isl_qpolynomial_fold_restore_list` (+3 more) 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理; 保留用于解释意图、用法或算法背景的注释.

### Lines 272-307

````c
/* Internal data structure for isl_qpolynomial_fold_*_dims
 * representing their arguments.
 */
struct isl_fold_dims_data {
	enum isl_dim_type type;
	unsigned first;
	unsigned n;
};

/* isl_qpolynomial_list_every callback that checks whether "qp"
 * does not involve any dimensions in the given range.
 */
static isl_bool not_involved(__isl_keep isl_qpolynomial *qp, void *user)
{
	struct isl_fold_dims_data *data = user;
	isl_bool involves;

	involves = isl_qpolynomial_involves_dims(qp, data->type,
							data->first, data->n);
	return isl_bool_not(involves);
}

/* Does "fold" involve any dimensions in the given range.
 *
 * It involves any of those dimensions if it is not the case
 * that every polynomial in the reduction does not involve
 * any of the dimensions.
 */
static isl_bool isl_qpolynomial_fold_involves_dims(
	__isl_keep isl_qpolynomial_fold *fold,
	enum isl_dim_type type, unsigned first, unsigned n)
{
	struct isl_fold_dims_data data = { type, first, n };
	isl_qpolynomial_list *list;
	isl_bool not;

````
- **EN**: This block declares or references types such as `isl_fold_dims_data`; defines enum values such as `isl_dim_type`; declares or defines routines around `not_involved`, `isl_qpolynomial_involves_dims`, `isl_qpolynomial_fold_involves_dims`; emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_fold_dims_data`; 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `not_involved`, `isl_qpolynomial_involves_dims`, `isl_qpolynomial_fold_involves_dims` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 308-334

````c
	if (!fold)
		return isl_bool_error;
	if (n == 0)
		return isl_bool_false;

	list = isl_qpolynomial_fold_peek_list(fold);
	not = isl_qpolynomial_list_every(list, &not_involved, &data);
	return isl_bool_not(not);
}

/* Internal data structure for isl_qpolynomial_fold_set_dim_name
 * representing its arguments.
 */
struct isl_fold_set_dim_name_data {
	enum isl_dim_type type;
	unsigned pos;
	const char *s;
};

/* isl_qpolynomial_list_map callback for calling
 * isl_qpolynomial_set_dim_name on "qp".
 */
static __isl_give isl_qpolynomial *set_dim_name(__isl_take isl_qpolynomial *qp,
	void *user)
{
	struct isl_fold_set_dim_name_data *data = user;

````
- **EN**: This block declares or references types such as `isl_fold_set_dim_name_data`; defines enum values such as `isl_dim_type`; declares or defines routines around `isl_qpolynomial_fold_peek_list`, `isl_qpolynomial_list_every`, `set_dim_name`; contains control flow with 2 conditional check(s); and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_fold_set_dim_name_data`; 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_qpolynomial_fold_peek_list`, `isl_qpolynomial_list_every`, `set_dim_name` 相关的例程; 包含控制流结构：2 处条件判断；并延续周边实现细节。

### Lines 335-361

````c
	qp = isl_qpolynomial_set_dim_name(qp, data->type, data->pos, data->s);
	return qp;
}

/* Given a dimension type for an isl_qpolynomial_fold,
 * return the corresponding type for the domain.
 */
static enum isl_dim_type domain_type(enum isl_dim_type type)
{
	if (type == isl_dim_in)
		return isl_dim_set;
	return type;
}

__isl_give isl_qpolynomial_fold *isl_qpolynomial_fold_set_dim_name(
	__isl_take isl_qpolynomial_fold *fold,
	enum isl_dim_type type, unsigned pos, const char *s)
{
	struct isl_fold_set_dim_name_data data = { type, pos, s };
	enum isl_dim_type set_type;
	isl_space *space;
	isl_qpolynomial_list *list;

	list = isl_qpolynomial_fold_take_list(fold);
	list = isl_qpolynomial_list_map(list, &set_dim_name, &data);
	fold = isl_qpolynomial_fold_restore_list(fold, list);

````
- **EN**: This block declares or references types such as `isl_fold_set_dim_name_data`; defines enum values such as `isl_dim_type`; declares or defines routines around `isl_qpolynomial_set_dim_name`, `domain_type`, `isl_qpolynomial_fold_set_dim_name`, `isl_qpolynomial_fold_take_list` (+2 more); contains control flow with 1 conditional check(s); and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_fold_set_dim_name_data`; 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_qpolynomial_set_dim_name`, `domain_type`, `isl_qpolynomial_fold_set_dim_name`, `isl_qpolynomial_fold_take_list` (+2 more) 相关的例程; 包含控制流结构：1 处条件判断；并延续周边实现细节。

### Lines 362-390

````c
	set_type = domain_type(type);
	space = isl_qpolynomial_fold_take_domain_space(fold);
	space = isl_space_set_dim_name(space, set_type, pos, s);
	fold = isl_qpolynomial_fold_restore_domain_space(fold, space);

	return fold;
}

/* isl_qpolynomial_list_map callback for calling
 * isl_qpolynomial_drop_dims on "qp".
 */
static __isl_give isl_qpolynomial *drop_dims(__isl_take isl_qpolynomial *qp,
	void *user)
{
	struct isl_fold_dims_data *data = user;

	qp = isl_qpolynomial_drop_dims(qp, data->type, data->first, data->n);
	return qp;
}

__isl_give isl_qpolynomial_fold *isl_qpolynomial_fold_drop_dims(
	__isl_take isl_qpolynomial_fold *fold,
	enum isl_dim_type type, unsigned first, unsigned n)
{
	struct isl_fold_dims_data data = { type, first, n };
	enum isl_dim_type set_type;
	isl_space *space;
	isl_qpolynomial_list *list;

````
- **EN**: This block declares or references types such as `isl_fold_dims_data`; defines enum values such as `isl_dim_type`; declares or defines routines around `domain_type`, `isl_qpolynomial_fold_take_domain_space`, `isl_space_set_dim_name`, `isl_qpolynomial_fold_restore_domain_space` (+3 more); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_fold_dims_data`; 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `domain_type`, `isl_qpolynomial_fold_take_domain_space`, `isl_space_set_dim_name`, `isl_qpolynomial_fold_restore_domain_space` (+3 more) 相关的例程; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 391-416

````c
	if (!fold)
		return NULL;
	if (n == 0)
		return fold;

	set_type = domain_type(type);

	list = isl_qpolynomial_fold_take_list(fold);
	list = isl_qpolynomial_list_map(list, &drop_dims, &data);
	fold = isl_qpolynomial_fold_restore_list(fold, list);

	space = isl_qpolynomial_fold_take_domain_space(fold);
	space = isl_space_drop_dims(space, set_type, first, n);
	fold = isl_qpolynomial_fold_restore_domain_space(fold, space);

	return fold;
}

/* isl_qpolynomial_list_map callback for calling
 * isl_qpolynomial_insert_dims on "qp".
 */
static __isl_give isl_qpolynomial *insert_dims(__isl_take isl_qpolynomial *qp,
	void *user)
{
	struct isl_fold_dims_data *data = user;

````
- **EN**: This block declares or references types such as `isl_fold_dims_data`; declares or defines routines around `domain_type`, `isl_qpolynomial_fold_take_list`, `isl_qpolynomial_list_map`, `isl_qpolynomial_fold_restore_list` (+4 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_fold_dims_data`; 声明或定义与 `domain_type`, `isl_qpolynomial_fold_take_list`, `isl_qpolynomial_list_map`, `isl_qpolynomial_fold_restore_list` (+4 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 417-443

````c
	qp = isl_qpolynomial_insert_dims(qp, data->type, data->first, data->n);
	return qp;
}

__isl_give isl_qpolynomial_fold *isl_qpolynomial_fold_insert_dims(
	__isl_take isl_qpolynomial_fold *fold,
	enum isl_dim_type type, unsigned first, unsigned n)
{
	struct isl_fold_dims_data data = { type, first, n };
	enum isl_dim_type set_type;
	isl_space *space;
	isl_qpolynomial_list *list;

	if (!fold)
		return NULL;
	if (n == 0 && !isl_space_is_named_or_nested(fold->dim, type))
		return fold;

	list = isl_qpolynomial_fold_take_list(fold);
	list = isl_qpolynomial_list_map(list, &insert_dims, &data);
	fold = isl_qpolynomial_fold_restore_list(fold, list);

	set_type = domain_type(type);
	space = isl_qpolynomial_fold_take_domain_space(fold);
	space = isl_space_insert_dims(space, set_type, first, n);
	fold = isl_qpolynomial_fold_restore_domain_space(fold, space);

````
- **EN**: This block declares or references types such as `isl_fold_dims_data`; defines enum values such as `isl_dim_type`; declares or defines routines around `isl_qpolynomial_insert_dims`, `isl_qpolynomial_fold_insert_dims`, `isl_qpolynomial_fold_take_list`, `isl_qpolynomial_list_map` (+5 more); contains control flow with 2 conditional check(s); and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_fold_dims_data`; 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_qpolynomial_insert_dims`, `isl_qpolynomial_fold_insert_dims`, `isl_qpolynomial_fold_take_list`, `isl_qpolynomial_list_map` (+5 more) 相关的例程; 包含控制流结构：2 处条件判断；并延续周边实现细节。

### Lines 444-470

````c
	return fold;
}

/* Determine the sign of the constant quasipolynomial "qp".
 *
 * Return
 *	-1 if qp <= 0
 *	 1 if qp >= 0
 *	 0 if unknown
 *
 * For qp == 0, we can return either -1 or 1.  In practice, we return 1.
 * For qp == NaN, the sign is undefined, so we return 0.
 */
static int isl_qpolynomial_cst_sign(__isl_keep isl_qpolynomial *qp)
{
	isl_poly_cst *cst;

	if (isl_qpolynomial_is_nan(qp))
		return 0;

	cst = isl_poly_as_cst(qp->poly);
	if (!cst)
		return 0;

	return isl_int_sgn(cst->n) < 0 ? -1 : 1;
}

````
- **EN**: This block declares or defines routines around `isl_qpolynomial_cst_sign`, `isl_poly_as_cst`; contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_qpolynomial_cst_sign`, `isl_poly_as_cst` 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 471-499

````c
static int isl_qpolynomial_aff_sign(__isl_keep isl_set *set,
	__isl_keep isl_qpolynomial *qp)
{
	enum isl_lp_result res;
	isl_vec *aff;
	isl_int opt;
	int sgn = 0;

	aff = isl_qpolynomial_extract_affine(qp);
	if (!aff)
		return 0;

	isl_int_init(opt);

	res = isl_set_solve_lp(set, 0, aff->el + 1, aff->el[0],
				&opt, NULL, NULL);
	if (res == isl_lp_error)
		goto done;
	if (res == isl_lp_empty ||
	    (res == isl_lp_ok && !isl_int_is_neg(opt))) {
		sgn = 1;
		goto done;
	}

	res = isl_set_solve_lp(set, 1, aff->el + 1, aff->el[0],
				&opt, NULL, NULL);
	if (res == isl_lp_ok && !isl_int_is_pos(opt))
		sgn = -1;

````
- **EN**: This block defines enum values such as `isl_lp_result`; declares or defines routines around `isl_qpolynomial_aff_sign`, `isl_qpolynomial_extract_affine`, `isl_int_init`, `isl_set_solve_lp` (+1 more); contains control flow with 4 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 定义枚举类型，例如 `isl_lp_result`; 声明或定义与 `isl_qpolynomial_aff_sign`, `isl_qpolynomial_extract_affine`, `isl_int_init`, `isl_set_solve_lp` (+1 more) 相关的例程; 包含控制流结构：4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 500-537

````c
done:
	isl_int_clear(opt);
	isl_vec_free(aff);
	return sgn;
}

/* Determine, if possible, the sign of the quasipolynomial "qp" on
 * the domain "set".
 *
 * If qp is a constant, then the problem is trivial.
 * If qp is linear, then we check if the minimum of the corresponding
 * affine constraint is non-negative or if the maximum is non-positive.
 *
 * Otherwise, we check if the outermost variable "v" has a lower bound "l"
 * in "set".  If so, we write qp(v,v') as
 *
 *	q(v,v') * (v - l) + r(v')
 *
 * if q(v,v') and r(v') have the same known sign, then the original
 * quasipolynomial has the same sign as well.
 *
 * Return
 *	-1 if qp <= 0
 *	 1 if qp >= 0
 *	 0 if unknown
 */
static int isl_qpolynomial_sign(__isl_keep isl_set *set,
	__isl_keep isl_qpolynomial *qp)
{
	isl_size d;
	int i;
	isl_bool is;
	isl_poly_rec *rec;
	isl_vec *v;
	isl_int l;
	enum isl_lp_result res;
	int sgn = 0;

````
- **EN**: This block defines enum values such as `isl_lp_result`; declares or defines routines around `isl_int_clear`, `isl_vec_free`, `qp`, `q` (+1 more); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 定义枚举类型，例如 `isl_lp_result`; 声明或定义与 `isl_int_clear`, `isl_vec_free`, `qp`, `q` (+1 more) 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 538-563

````c
	is = isl_qpolynomial_is_cst(qp, NULL, NULL);
	if (is < 0)
		return 0;
	if (is)
		return isl_qpolynomial_cst_sign(qp);

	is = isl_qpolynomial_is_affine(qp);
	if (is < 0)
		return 0;
	if (is)
		return isl_qpolynomial_aff_sign(set, qp);

	if (qp->div->n_row > 0)
		return 0;

	rec = isl_poly_as_rec(qp->poly);
	if (!rec)
		return 0;

	d = isl_space_dim(qp->dim, isl_dim_all);
	if (d < 0)
		return 0;
	v = isl_vec_alloc(set->ctx, 2 + d);
	if (!v)
		return 0;

````
- **EN**: This block declares or defines routines around `isl_qpolynomial_is_cst`, `isl_qpolynomial_is_affine`, `isl_poly_as_rec`, `isl_space_dim` (+1 more); contains control flow with 8 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_qpolynomial_is_cst`, `isl_qpolynomial_is_affine`, `isl_poly_as_rec`, `isl_space_dim` (+1 more) 相关的例程; 包含控制流结构：8 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 564-595

````c
	isl_seq_clr(v->el + 1, 1 + d);
	isl_int_set_si(v->el[0], 1);
	isl_int_set_si(v->el[2 + qp->poly->var], 1);

	isl_int_init(l);

	res = isl_set_solve_lp(set, 0, v->el + 1, v->el[0], &l, NULL, NULL);
	if (res == isl_lp_ok) {
		isl_qpolynomial *min;
		isl_qpolynomial *base;
		isl_qpolynomial *r, *q;
		isl_qpolynomial *t;

		min = isl_qpolynomial_cst_on_domain(isl_space_copy(qp->dim), l);
		base = isl_qpolynomial_var_pow_on_domain(isl_space_copy(qp->dim),
						qp->poly->var, 1);

		r = isl_qpolynomial_alloc(isl_space_copy(qp->dim), 0,
					  isl_poly_copy(rec->p[rec->n - 1]));
		q = isl_qpolynomial_copy(r);

		for (i = rec->n - 2; i >= 0; --i) {
			r = isl_qpolynomial_mul(r, isl_qpolynomial_copy(min));
			t = isl_qpolynomial_alloc(isl_space_copy(qp->dim), 0,
						  isl_poly_copy(rec->p[i]));
			r = isl_qpolynomial_add(r, t);
			if (i == 0)
				break;
			q = isl_qpolynomial_mul(q, isl_qpolynomial_copy(base));
			q = isl_qpolynomial_add(q, isl_qpolynomial_copy(r));
		}

````
- **EN**: This block declares or defines routines around `isl_seq_clr`, `isl_int_set_si`, `isl_int_init`, `isl_set_solve_lp` (+7 more); contains control flow with 1 loop construct(s), 2 conditional check(s); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_seq_clr`, `isl_int_set_si`, `isl_int_init`, `isl_set_solve_lp` (+7 more) 相关的例程; 包含控制流结构：1 处循环、2 处条件判断; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 596-628

````c
		if (isl_qpolynomial_is_zero(q))
			sgn = isl_qpolynomial_sign(set, r);
		else if (isl_qpolynomial_is_zero(r))
			sgn = isl_qpolynomial_sign(set, q);
		else {
			int sgn_q, sgn_r;
			sgn_r = isl_qpolynomial_sign(set, r);
			sgn_q = isl_qpolynomial_sign(set, q);
			if (sgn_r == sgn_q)
				sgn = sgn_r;
		}

		isl_qpolynomial_free(min);
		isl_qpolynomial_free(base);
		isl_qpolynomial_free(q);
		isl_qpolynomial_free(r);
	}

	isl_int_clear(l);

	isl_vec_free(v);

	return sgn;
}

/* Check that "fold1" and "fold2" have the same type.
 */
static isl_stat isl_qpolynomial_fold_check_equal_type(
	__isl_keep isl_qpolynomial_fold *fold1,
	__isl_keep isl_qpolynomial_fold *fold2)
{
	enum isl_fold type1, type2;

````
- **EN**: This block defines enum values such as `isl_fold`; declares or defines routines around `isl_qpolynomial_sign`, `isl_qpolynomial_free`, `isl_int_clear`, `isl_vec_free` (+1 more); contains control flow with 3 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 定义枚举类型，例如 `isl_fold`; 声明或定义与 `isl_qpolynomial_sign`, `isl_qpolynomial_free`, `isl_int_clear`, `isl_vec_free` (+1 more) 相关的例程; 包含控制流结构：3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 629-658

````c
	type1 = isl_qpolynomial_fold_get_type(fold1);
	type2 = isl_qpolynomial_fold_get_type(fold2);
	if (type1 < 0 || type2 < 0)
		return isl_stat_error;
	if (type1 != type2)
		isl_die(isl_qpolynomial_fold_get_ctx(fold1), isl_error_invalid,
			"fold types don't match", return isl_stat_error);
	return isl_stat_ok;
}

/* Check that "fold1" and "fold2" have the same (domain) space.
 */
static isl_stat isl_qpolynomial_fold_check_equal_space(
	__isl_keep isl_qpolynomial_fold *fold1,
	__isl_keep isl_qpolynomial_fold *fold2)
{
	isl_bool equal;
	isl_space *space1, *space2;

	space1 = isl_qpolynomial_fold_peek_domain_space(fold1);
	space2 = isl_qpolynomial_fold_peek_domain_space(fold2);
	equal = isl_space_is_equal(space1, space2);
	if (equal < 0)
		return isl_stat_error;
	if (!equal)
		isl_die(isl_qpolynomial_fold_get_ctx(fold1), isl_error_invalid,
			"spaces don't match", return isl_stat_error);
	return isl_stat_ok;
}

````
- **EN**: This block declares or defines routines around `isl_qpolynomial_fold_get_type`, `isl_die`, `same`, `isl_qpolynomial_fold_check_equal_space` (+2 more); contains control flow with 4 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_qpolynomial_fold_get_type`, `isl_die`, `same`, `isl_qpolynomial_fold_check_equal_space` (+2 more) 相关的例程; 包含控制流结构：4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 659-690

````c
/* Combine "list1" and "list2" into a single list, eliminating
 * those elements of one list that are already covered by the other
 * list on "set".
 *
 * "better" is the sign that the difference qp1 - qp2 needs to have for qp1
 * to be covered by qp2.
 */
static __isl_give isl_qpolynomial_list *merge_lists(__isl_keep isl_set *set,
	__isl_take isl_qpolynomial_list *list1,
	__isl_take isl_qpolynomial_list *list2, int better)
{
	int i, j;
	isl_size n1, n2;

	n1 = isl_qpolynomial_list_size(list1);
	n2 = isl_qpolynomial_list_size(list2);
	if (n1 < 0 || n2 < 0)
		goto error;

	for (i = n2 - 1; i >= 0; --i) {
		for (j = n1 - 1; j >= 0; --j) {
			isl_qpolynomial *qp1, *qp2, *d;
			int sgn;
			isl_bool equal;

			qp1 = isl_qpolynomial_list_peek(list1, j);
			qp2 = isl_qpolynomial_list_peek(list2, i);
			equal = isl_qpolynomial_plain_is_equal(qp1, qp2);
			if (equal < 0)
				goto error;
			if (equal)
				break;
````
- **EN**: This block declares or defines routines around `merge_lists`, `isl_qpolynomial_list_size`, `isl_qpolynomial_list_peek`, `isl_qpolynomial_plain_is_equal`; contains control flow with 2 loop construct(s), 3 conditional check(s); uses ISL data structures or helpers for polyhedral reasoning; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `merge_lists`, `isl_qpolynomial_list_size`, `isl_qpolynomial_list_peek`, `isl_qpolynomial_plain_is_equal` 相关的例程; 包含控制流结构：2 处循环、3 处条件判断; 使用 ISL 数据结构或辅助函数进行多面体推理; 保留用于解释意图、用法或算法背景的注释.

### Lines 691-722

````c
			d = isl_qpolynomial_sub(
				isl_qpolynomial_copy(qp1),
				isl_qpolynomial_copy(qp2));
			sgn = isl_qpolynomial_sign(set, d);
			isl_qpolynomial_free(d);
			if (sgn == 0)
				continue;
			if (sgn != better)
				break;
			list1 = isl_qpolynomial_list_drop(list1, j, 1);
			n1--;
		}
		if (j < 0)
			continue;
		list2 = isl_qpolynomial_list_drop(list2, i, 1);
		n2--;
	}

	return isl_qpolynomial_list_concat(list1, list2);
error:
	isl_qpolynomial_list_free(list1);
	isl_qpolynomial_list_free(list2);
	return NULL;
}

/* Combine "fold1" and "fold2" into a single reduction, eliminating
 * those elements of one reduction that are already covered by the other
 * reduction on "set".
 *
 * If "fold1" or "fold2" is an empty reduction, then return
 * the other reduction.
 * If "fold1" or "fold2" is a NaN, then return this NaN.
````
- **EN**: This block declares or defines routines around `isl_qpolynomial_sub`, `isl_qpolynomial_copy`, `isl_qpolynomial_sign`, `isl_qpolynomial_free` (+2 more); contains control flow with 3 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_qpolynomial_sub`, `isl_qpolynomial_copy`, `isl_qpolynomial_sign`, `isl_qpolynomial_free` (+2 more) 相关的例程; 包含控制流结构：3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 723-751

````c
 */
__isl_give isl_qpolynomial_fold *isl_qpolynomial_fold_fold_on_domain(
	__isl_keep isl_set *set,
	__isl_take isl_qpolynomial_fold *fold1,
	__isl_take isl_qpolynomial_fold *fold2)
{
	isl_qpolynomial_list *list1;
	isl_qpolynomial_list *list2;
	int better;

	if (isl_qpolynomial_fold_check_equal_type(fold1, fold2) < 0)
		goto error;
	if (isl_qpolynomial_fold_check_equal_space(fold1, fold2) < 0)
		goto error;

	better = fold1->type == isl_fold_max ? -1 : 1;

	if (isl_qpolynomial_fold_is_empty(fold1) ||
	    isl_qpolynomial_fold_is_nan(fold2)) {
		isl_qpolynomial_fold_free(fold1);
		return fold2;
	}

	if (isl_qpolynomial_fold_is_empty(fold2) ||
	    isl_qpolynomial_fold_is_nan(fold1)) {
		isl_qpolynomial_fold_free(fold2);
		return fold1;
	}

````
- **EN**: This block declares or defines routines around `isl_qpolynomial_fold_fold_on_domain`, `isl_qpolynomial_fold_is_nan`, `isl_qpolynomial_fold_free`; contains control flow with 4 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_qpolynomial_fold_fold_on_domain`, `isl_qpolynomial_fold_is_nan`, `isl_qpolynomial_fold_free` 相关的例程; 包含控制流结构：4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 752-781

````c
	list1 = isl_qpolynomial_fold_take_list(fold1);
	list2 = isl_qpolynomial_fold_take_list(fold2);

	list1 = merge_lists(set, list1, list2, better);

	fold1 = isl_qpolynomial_fold_restore_list(fold1, list1);
	isl_qpolynomial_fold_free(fold2);

	return fold1;
error:
	isl_qpolynomial_fold_free(fold1);
	isl_qpolynomial_fold_free(fold2);
	return NULL;
}

/* isl_qpolynomial_list_map callback for adding "qp2" to "qp".
 */
static __isl_give isl_qpolynomial *add_qpolynomial(
	__isl_take isl_qpolynomial *qp, void *user)
{
	isl_qpolynomial *qp2 = user;

	return isl_qpolynomial_add(qp, isl_qpolynomial_copy(qp2));
}

__isl_give isl_qpolynomial_fold *isl_qpolynomial_fold_add_qpolynomial(
	__isl_take isl_qpolynomial_fold *fold, __isl_take isl_qpolynomial *qp)
{
	isl_qpolynomial_list *list;

````
- **EN**: This block declares or defines routines around `isl_qpolynomial_fold_take_list`, `merge_lists`, `isl_qpolynomial_fold_restore_list`, `isl_qpolynomial_fold_free` (+2 more); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `isl_qpolynomial_fold_take_list`, `merge_lists`, `isl_qpolynomial_fold_restore_list`, `isl_qpolynomial_fold_free` (+2 more) 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理; 保留用于解释意图、用法或算法背景的注释.

### Lines 782-812

````c
	if (!fold || !qp)
		goto error;

	if (isl_qpolynomial_is_zero(qp)) {
		isl_qpolynomial_free(qp);
		return fold;
	}

	list = isl_qpolynomial_fold_take_list(fold);
	list = isl_qpolynomial_list_map(list, &add_qpolynomial, qp);
	fold = isl_qpolynomial_fold_restore_list(fold, list);

	isl_qpolynomial_free(qp);
	return fold;
error:
	isl_qpolynomial_fold_free(fold);
	isl_qpolynomial_free(qp);
	return NULL;
}

__isl_give isl_qpolynomial_fold *isl_qpolynomial_fold_add_on_domain(
	__isl_keep isl_set *dom,
	__isl_take isl_qpolynomial_fold *fold1,
	__isl_take isl_qpolynomial_fold *fold2)
{
	int i;
	isl_size n1, n2;
	isl_qpolynomial_fold *res = NULL;
	isl_qpolynomial *qp;
	isl_qpolynomial_list *list1, *list2;

````
- **EN**: This block declares or defines routines around `isl_qpolynomial_free`, `isl_qpolynomial_fold_take_list`, `isl_qpolynomial_list_map`, `isl_qpolynomial_fold_restore_list` (+2 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_qpolynomial_free`, `isl_qpolynomial_fold_take_list`, `isl_qpolynomial_list_map`, `isl_qpolynomial_fold_restore_list` (+2 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 813-842

````c
	if (!fold1 || !fold2)
		goto error;

	if (isl_qpolynomial_fold_is_empty(fold1)) {
		isl_qpolynomial_fold_free(fold1);
		return fold2;
	}

	if (isl_qpolynomial_fold_is_empty(fold2)) {
		isl_qpolynomial_fold_free(fold2);
		return fold1;
	}

	list1 = isl_qpolynomial_fold_peek_list(fold1);
	list2 = isl_qpolynomial_fold_peek_list(fold2);
	n1 = isl_qpolynomial_list_size(list1);
	n2 = isl_qpolynomial_list_size(list2);
	if (n1 < 0 || n2 < 0)
		goto error;

	if (n1 == 1 && n2 != 1)
		return isl_qpolynomial_fold_add_on_domain(dom, fold2, fold1);

	qp = isl_qpolynomial_list_get_at(list2, 0);
	if (n2 == 1) {
		res = isl_qpolynomial_fold_add_qpolynomial(fold1, qp);
		isl_qpolynomial_fold_free(fold2);
		return res;
	}

````
- **EN**: This block declares or defines routines around `isl_qpolynomial_fold_free`, `isl_qpolynomial_fold_peek_list`, `isl_qpolynomial_list_size`, `isl_qpolynomial_list_get_at` (+1 more); contains control flow with 6 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_qpolynomial_fold_free`, `isl_qpolynomial_fold_peek_list`, `isl_qpolynomial_list_size`, `isl_qpolynomial_list_get_at` (+1 more) 相关的例程; 包含控制流结构：6 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 843-872

````c
	res = isl_qpolynomial_fold_add_qpolynomial(
				isl_qpolynomial_fold_copy(fold1), qp);

	for (i = 1; i < n2; ++i) {
		isl_qpolynomial_fold *res_i;

		qp = isl_qpolynomial_list_get_at(list2, i);
		res_i = isl_qpolynomial_fold_add_qpolynomial(
					isl_qpolynomial_fold_copy(fold1), qp);
		res = isl_qpolynomial_fold_fold_on_domain(dom, res, res_i);
	}

	isl_qpolynomial_fold_free(fold1);
	isl_qpolynomial_fold_free(fold2);
	return res;
error:
	isl_qpolynomial_fold_free(res);
	isl_qpolynomial_fold_free(fold1);
	isl_qpolynomial_fold_free(fold2);
	return NULL;
}

/* isl_qpolynomial_list_map callback for calling
 * isl_qpolynomial_substitute_equalities on "qp" and "eq".
 */
static __isl_give isl_qpolynomial *substitute_equalities(
	__isl_take isl_qpolynomial *qp, void *user)
{
	isl_basic_set *eq = user;

````
- **EN**: This block declares or defines routines around `isl_qpolynomial_fold_add_qpolynomial`, `isl_qpolynomial_fold_copy`, `isl_qpolynomial_list_get_at`, `isl_qpolynomial_fold_fold_on_domain` (+2 more); contains control flow with 1 loop construct(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_qpolynomial_fold_add_qpolynomial`, `isl_qpolynomial_fold_copy`, `isl_qpolynomial_list_get_at`, `isl_qpolynomial_fold_fold_on_domain` (+2 more) 相关的例程; 包含控制流结构：1 处循环; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 873-900

````c
	eq = isl_basic_set_copy(eq);
	return isl_qpolynomial_substitute_equalities(qp, eq);
}

__isl_give isl_qpolynomial_fold *isl_qpolynomial_fold_substitute_equalities(
	__isl_take isl_qpolynomial_fold *fold, __isl_take isl_basic_set *eq)
{
	isl_qpolynomial_list *list;

	list = isl_qpolynomial_fold_take_list(fold);
	list = isl_qpolynomial_list_map(list, &substitute_equalities, eq);
	fold = isl_qpolynomial_fold_restore_list(fold, list);

	isl_basic_set_free(eq);
	return fold;
}

/* isl_qpolynomial_list_map callback for calling
 * isl_qpolynomial_substitute_equalities on "qp" and "context".
 */
static __isl_give isl_qpolynomial *gist(__isl_take isl_qpolynomial *qp,
	void *user)
{
	isl_set *context = user;

	return isl_qpolynomial_gist(qp, isl_set_copy(context));
}

````
- **EN**: This block declares or defines routines around `isl_basic_set_copy`, `isl_qpolynomial_fold_substitute_equalities`, `isl_qpolynomial_fold_take_list`, `isl_qpolynomial_list_map` (+3 more); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `isl_basic_set_copy`, `isl_qpolynomial_fold_substitute_equalities`, `isl_qpolynomial_fold_take_list`, `isl_qpolynomial_list_map` (+3 more) 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理; 保留用于解释意图、用法或算法背景的注释.

### Lines 901-933

````c
__isl_give isl_qpolynomial_fold *isl_qpolynomial_fold_gist(
	__isl_take isl_qpolynomial_fold *fold, __isl_take isl_set *context)
{
	isl_qpolynomial_list *list;

	list = isl_qpolynomial_fold_take_list(fold);
	list = isl_qpolynomial_list_map(list, &gist, context);
	fold = isl_qpolynomial_fold_restore_list(fold, list);

	isl_set_free(context);
	return fold;
}

__isl_give isl_qpolynomial_fold *isl_qpolynomial_fold_gist_params(
	__isl_take isl_qpolynomial_fold *fold, __isl_take isl_set *context)
{
	isl_space *space = isl_qpolynomial_fold_get_domain_space(fold);
	isl_set *dom_context = isl_set_universe(space);
	dom_context = isl_set_intersect_params(dom_context, context);
	return isl_qpolynomial_fold_gist(fold, dom_context);
}

/* Return a zero (i.e., empty) isl_qpolynomial_fold in the given space.
 *
 * This is a helper function for isl_pw_*_as_* that ensures a uniform
 * interface over all piecewise types.
 */
static __isl_give isl_qpolynomial_fold *isl_qpolynomial_fold_zero_in_space(
	__isl_take isl_space *space, enum isl_fold type)
{
	return isl_qpolynomial_fold_empty(type, isl_space_domain(space));
}

````
- **EN**: This block declares or defines routines around `isl_qpolynomial_fold_gist`, `isl_qpolynomial_fold_take_list`, `isl_qpolynomial_list_map`, `isl_qpolynomial_fold_restore_list` (+7 more); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `isl_qpolynomial_fold_gist`, `isl_qpolynomial_fold_take_list`, `isl_qpolynomial_list_map`, `isl_qpolynomial_fold_restore_list` (+7 more) 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理; 保留用于解释意图、用法或算法背景的注释.

### Lines 934-963

````c
#define isl_qpolynomial_fold_involves_nan isl_qpolynomial_fold_is_nan

#define HAS_TYPE

#undef PW
#define PW isl_pw_qpolynomial_fold
#undef BASE
#define BASE qpolynomial_fold
#undef EL_IS_ZERO
#define EL_IS_ZERO is_empty
#undef ZERO
#define ZERO zero
#undef IS_ZERO
#define IS_ZERO is_zero
#undef FIELD
#define FIELD fold
#undef DEFAULT_IS_ZERO
#define DEFAULT_IS_ZERO 1

#include <isl_pw_templ.c>
#include <isl_pw_add_disjoint_templ.c>
#include <isl_pw_eval.c>
#include <isl_pw_fix_templ.c>
#include <isl_pw_from_range_templ.c>
#include <isl_pw_insert_dims_templ.c>
#include <isl_pw_lift_templ.c>
#include <isl_pw_morph_templ.c>
#include <isl_pw_move_dims_templ.c>
#include <isl_pw_opt_templ.c>

````
- **EN**: This block imports system/standard headers needed by the surrounding code; uses preprocessor directives to control compilation, macros, or include guards; defines macros like `isl_qpolynomial_fold_involves_nan`, `HAS_TYPE`, `PW`, `BASE` (+5 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 引入周边逻辑所需的 system/standard 头文件; 使用预处理指令控制编译、宏或头文件保护; 定义宏，例如 `isl_qpolynomial_fold_involves_nan`, `HAS_TYPE`, `PW`, `BASE` (+5 more); 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 964-994

````c
#undef BASE
#define BASE pw_qpolynomial_fold

#include <isl_union_single.c>
#include <isl_union_eval.c>

/* Construct a new reduction of the given type and space
 * with an empty list of polynomials.
 */
__isl_give isl_qpolynomial_fold *isl_qpolynomial_fold_empty(enum isl_fold type,
	__isl_take isl_space *space)
{
	isl_ctx *ctx;
	isl_qpolynomial_list *list;

	if (!space)
		return NULL;
	ctx = isl_space_get_ctx(space);
	list = isl_qpolynomial_list_alloc(ctx, 0);
	return qpolynomial_fold_alloc(type, space, list);
}

/* Construct a new reduction of the given type and
 * a single given polynomial.
 */
__isl_give isl_qpolynomial_fold *isl_qpolynomial_fold_alloc(
	enum isl_fold type, __isl_take isl_qpolynomial *qp)
{
	isl_space *space;
	isl_qpolynomial_list *list;

````
- **EN**: This block imports system/standard headers needed by the surrounding code; uses preprocessor directives to control compilation, macros, or include guards; defines enum values such as `isl_fold`; defines macros like `BASE`; and continues the surrounding implementation details.
- **CN**: 该代码块 引入周边逻辑所需的 system/standard 头文件; 使用预处理指令控制编译、宏或头文件保护; 定义枚举类型，例如 `isl_fold`; 定义宏，例如 `BASE`；并延续周边实现细节。

### Lines 995-1022

````c
	space = isl_qpolynomial_get_domain_space(qp);
	list = isl_qpolynomial_list_from_qpolynomial(qp);
	return qpolynomial_fold_alloc(type, space, list);
}

__isl_give isl_qpolynomial_fold *isl_qpolynomial_fold_copy(
	__isl_keep isl_qpolynomial_fold *fold)
{
	if (!fold)
		return NULL;

	fold->ref++;
	return fold;
}

__isl_give isl_qpolynomial_fold *isl_qpolynomial_fold_dup(
	__isl_keep isl_qpolynomial_fold *fold)
{
	enum isl_fold type;
	isl_space *space;
	isl_qpolynomial_list *list;

	type = isl_qpolynomial_fold_get_type(fold);
	space = isl_qpolynomial_fold_get_domain_space(fold);
	list = isl_qpolynomial_fold_get_list(fold);
	return qpolynomial_fold_alloc(type, space, list);
}

````
- **EN**: This block defines enum values such as `isl_fold`; declares or defines routines around `isl_qpolynomial_get_domain_space`, `isl_qpolynomial_list_from_qpolynomial`, `isl_qpolynomial_fold_copy`, `isl_qpolynomial_fold_dup` (+3 more); contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 定义枚举类型，例如 `isl_fold`; 声明或定义与 `isl_qpolynomial_get_domain_space`, `isl_qpolynomial_list_from_qpolynomial`, `isl_qpolynomial_fold_copy`, `isl_qpolynomial_fold_dup` (+3 more) 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 1023-1049

````c
__isl_give isl_qpolynomial_fold *isl_qpolynomial_fold_cow(
	__isl_take isl_qpolynomial_fold *fold)
{
	if (!fold)
		return NULL;

	if (fold->ref == 1)
		return fold;
	fold->ref--;
	return isl_qpolynomial_fold_dup(fold);
}

__isl_null isl_qpolynomial_fold *isl_qpolynomial_fold_free(
	__isl_take isl_qpolynomial_fold *fold)
{
	if (!fold)
		return NULL;
	if (--fold->ref > 0)
		return NULL;

	isl_qpolynomial_list_free(fold->list);
	isl_space_free(fold->dim);
	free(fold);

	return NULL;
}

````
- **EN**: This block declares or defines routines around `isl_qpolynomial_fold_cow`, `isl_qpolynomial_fold_free`, `isl_qpolynomial_list_free`, `isl_space_free` (+1 more); contains control flow with 4 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_qpolynomial_fold_cow`, `isl_qpolynomial_fold_free`, `isl_qpolynomial_list_free`, `isl_space_free` (+1 more) 相关的例程; 包含控制流结构：4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1050-1080

````c
isl_bool isl_qpolynomial_fold_is_empty(__isl_keep isl_qpolynomial_fold *fold)
{
	isl_size n;
	isl_qpolynomial_list *list;

	list = isl_qpolynomial_fold_peek_list(fold);
	n = isl_qpolynomial_list_size(list);
	if (n < 0)
		return isl_bool_error;

	return isl_bool_ok(n == 0);
}

/* Does "fold" represent max(NaN) or min(NaN)?
 */
isl_bool isl_qpolynomial_fold_is_nan(__isl_keep isl_qpolynomial_fold *fold)
{
	isl_size n;
	isl_qpolynomial *qp;
	isl_qpolynomial_list *list;

	list = isl_qpolynomial_fold_peek_list(fold);
	n = isl_qpolynomial_list_size(list);
	if (n < 0)
		return isl_bool_error;
	if (n != 1)
		return isl_bool_false;
	qp = isl_qpolynomial_list_peek(list, 0);
	return isl_qpolynomial_is_nan(qp);
}

````
- **EN**: This block declares or defines routines around `isl_qpolynomial_fold_is_empty`, `isl_qpolynomial_fold_peek_list`, `isl_qpolynomial_list_size`, `max` (+2 more); contains control flow with 3 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_qpolynomial_fold_is_empty`, `isl_qpolynomial_fold_peek_list`, `isl_qpolynomial_list_size`, `max` (+2 more) 相关的例程; 包含控制流结构：3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 1081-1107

````c
__isl_give isl_qpolynomial_fold *isl_qpolynomial_fold_fold(
	__isl_take isl_qpolynomial_fold *fold1,
	__isl_take isl_qpolynomial_fold *fold2)
{
	isl_qpolynomial_list *list1, *list2;

	if (isl_qpolynomial_fold_check_equal_type(fold1, fold2) < 0)
		goto error;
	if (isl_qpolynomial_fold_check_equal_space(fold1, fold2) < 0)
		goto error;

	if (isl_qpolynomial_fold_is_empty(fold1)) {
		isl_qpolynomial_fold_free(fold1);
		return fold2;
	}

	if (isl_qpolynomial_fold_is_empty(fold2)) {
		isl_qpolynomial_fold_free(fold2);
		return fold1;
	}

	list1 = isl_qpolynomial_fold_take_list(fold1);
	list2 = isl_qpolynomial_fold_take_list(fold2);
	list1 = isl_qpolynomial_list_concat(list1, list2);
	fold1 = isl_qpolynomial_fold_restore_list(fold1, list1);
	isl_qpolynomial_fold_free(fold2);

````
- **EN**: This block declares or defines routines around `isl_qpolynomial_fold_fold`, `isl_qpolynomial_fold_free`, `isl_qpolynomial_fold_take_list`, `isl_qpolynomial_list_concat` (+1 more); contains control flow with 4 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_qpolynomial_fold_fold`, `isl_qpolynomial_fold_free`, `isl_qpolynomial_fold_take_list`, `isl_qpolynomial_list_concat` (+1 more) 相关的例程; 包含控制流结构：4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1108-1137

````c
	return fold1;
error:
	isl_qpolynomial_fold_free(fold1);
	isl_qpolynomial_fold_free(fold2);
	return NULL;
}

__isl_give isl_pw_qpolynomial_fold *isl_pw_qpolynomial_fold_fold(
	__isl_take isl_pw_qpolynomial_fold *pw1,
	__isl_take isl_pw_qpolynomial_fold *pw2)
{
	int i, j, n;
	struct isl_pw_qpolynomial_fold *res;
	isl_set *set;

	if (!pw1 || !pw2)
		goto error;

	isl_assert(pw1->dim->ctx, isl_space_is_equal(pw1->dim, pw2->dim), goto error);

	if (isl_pw_qpolynomial_fold_is_zero(pw1)) {
		isl_pw_qpolynomial_fold_free(pw1);
		return pw2;
	}

	if (isl_pw_qpolynomial_fold_is_zero(pw2)) {
		isl_pw_qpolynomial_fold_free(pw2);
		return pw1;
	}

````
- **EN**: This block declares or references types such as `isl_pw_qpolynomial_fold`; declares or defines routines around `isl_qpolynomial_fold_free`, `isl_pw_qpolynomial_fold_fold`, `isl_assert`, `isl_pw_qpolynomial_fold_free`; contains control flow with 3 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_pw_qpolynomial_fold`; 声明或定义与 `isl_qpolynomial_fold_free`, `isl_pw_qpolynomial_fold_fold`, `isl_assert`, `isl_pw_qpolynomial_fold_free` 相关的例程; 包含控制流结构：3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 1138-1163

````c
	if (pw1->type != pw2->type)
		isl_die(pw1->dim->ctx, isl_error_invalid,
			"fold types don't match", goto error);

	n = (pw1->n + 1) * (pw2->n + 1);
	res = isl_pw_qpolynomial_fold_alloc_size(isl_space_copy(pw1->dim),
						pw1->type, n);

	for (i = 0; i < pw1->n; ++i) {
		set = isl_set_copy(pw1->p[i].set);
		for (j = 0; j < pw2->n; ++j) {
			struct isl_set *common;
			isl_qpolynomial_fold *sum;
			set = isl_set_subtract(set,
					isl_set_copy(pw2->p[j].set));
			common = isl_set_intersect(isl_set_copy(pw1->p[i].set),
						isl_set_copy(pw2->p[j].set));
			if (isl_set_plain_is_empty(common)) {
				isl_set_free(common);
				continue;
			}

			sum = isl_qpolynomial_fold_fold_on_domain(common,
			       isl_qpolynomial_fold_copy(pw1->p[i].fold),
			       isl_qpolynomial_fold_copy(pw2->p[j].fold));

````
- **EN**: This block declares or references types such as `isl_set`; declares or defines routines around `isl_die`, `isl_pw_qpolynomial_fold_alloc_size`, `isl_set_copy`, `isl_set_subtract` (+4 more); contains control flow with 2 loop construct(s), 2 conditional check(s); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或引用类型，例如 `isl_set`; 声明或定义与 `isl_die`, `isl_pw_qpolynomial_fold_alloc_size`, `isl_set_copy`, `isl_set_subtract` (+4 more) 相关的例程; 包含控制流结构：2 处循环、2 处条件判断; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1164-1193

````c
			res = isl_pw_qpolynomial_fold_add_piece(res, common, sum);
		}
		res = isl_pw_qpolynomial_fold_add_piece(res, set,
			isl_qpolynomial_fold_copy(pw1->p[i].fold));
	}

	for (j = 0; j < pw2->n; ++j) {
		set = isl_set_copy(pw2->p[j].set);
		for (i = 0; i < pw1->n; ++i)
			set = isl_set_subtract(set, isl_set_copy(pw1->p[i].set));
		res = isl_pw_qpolynomial_fold_add_piece(res, set,
				    isl_qpolynomial_fold_copy(pw2->p[j].fold));
	}

	isl_pw_qpolynomial_fold_free(pw1);
	isl_pw_qpolynomial_fold_free(pw2);

	return res;
error:
	isl_pw_qpolynomial_fold_free(pw1);
	isl_pw_qpolynomial_fold_free(pw2);
	return NULL;
}

__isl_give isl_union_pw_qpolynomial_fold *isl_union_pw_qpolynomial_fold_fold_pw_qpolynomial_fold(
	__isl_take isl_union_pw_qpolynomial_fold *u,
	__isl_take isl_pw_qpolynomial_fold *part)
{
	struct isl_hash_table_entry *entry;

````
- **EN**: This block declares or references types such as `isl_hash_table_entry`; declares or defines routines around `isl_pw_qpolynomial_fold_add_piece`, `isl_qpolynomial_fold_copy`, `isl_set_copy`, `isl_set_subtract` (+2 more); contains control flow with 2 loop construct(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_hash_table_entry`; 声明或定义与 `isl_pw_qpolynomial_fold_add_piece`, `isl_qpolynomial_fold_copy`, `isl_set_copy`, `isl_set_subtract` (+2 more) 相关的例程; 包含控制流结构：2 处循环; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 1194-1221

````c
	u = isl_union_pw_qpolynomial_fold_cow(u);

	if (!part || !u)
		goto error;
	if (isl_space_check_equal_params(part->dim, u->space) < 0)
		goto error;

	entry = isl_union_pw_qpolynomial_fold_find_part_entry(u, part->dim, 1);
	if (!entry)
		goto error;

	if (!entry->data)
		entry->data = part;
	else {
		entry->data = isl_pw_qpolynomial_fold_fold(entry->data,
					    isl_pw_qpolynomial_fold_copy(part));
		if (!entry->data)
			goto error;
		isl_pw_qpolynomial_fold_free(part);
	}

	return u;
error:
	isl_pw_qpolynomial_fold_free(part);
	isl_union_pw_qpolynomial_fold_free(u);
	return NULL;
}

````
- **EN**: This block declares or defines routines around `isl_union_pw_qpolynomial_fold_cow`, `isl_union_pw_qpolynomial_fold_find_part_entry`, `isl_pw_qpolynomial_fold_fold`, `isl_pw_qpolynomial_fold_copy` (+2 more); contains control flow with 5 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_union_pw_qpolynomial_fold_cow`, `isl_union_pw_qpolynomial_fold_find_part_entry`, `isl_pw_qpolynomial_fold_fold`, `isl_pw_qpolynomial_fold_copy` (+2 more) 相关的例程; 包含控制流结构：5 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1222-1253

````c
static isl_stat fold_part(__isl_take isl_pw_qpolynomial_fold *part, void *user)
{
	isl_union_pw_qpolynomial_fold **u;
	u = (isl_union_pw_qpolynomial_fold **)user;

	*u = isl_union_pw_qpolynomial_fold_fold_pw_qpolynomial_fold(*u, part);

	return isl_stat_ok;
}

__isl_give isl_union_pw_qpolynomial_fold *isl_union_pw_qpolynomial_fold_fold(
	__isl_take isl_union_pw_qpolynomial_fold *u1,
	__isl_take isl_union_pw_qpolynomial_fold *u2)
{
	u1 = isl_union_pw_qpolynomial_fold_cow(u1);

	if (!u1 || !u2)
		goto error;

	if (isl_union_pw_qpolynomial_fold_foreach_pw_qpolynomial_fold(u2,
							&fold_part, &u1) < 0)
		goto error;

	isl_union_pw_qpolynomial_fold_free(u2);

	return u1;
error:
	isl_union_pw_qpolynomial_fold_free(u1);
	isl_union_pw_qpolynomial_fold_free(u2);
	return NULL;
}

````
- **EN**: This block declares or defines routines around `fold_part`, `isl_union_pw_qpolynomial_fold_fold_pw_qpolynomial_fold`, `isl_union_pw_qpolynomial_fold_fold`, `isl_union_pw_qpolynomial_fold_cow` (+1 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `fold_part`, `isl_union_pw_qpolynomial_fold_fold_pw_qpolynomial_fold`, `isl_union_pw_qpolynomial_fold_fold`, `isl_union_pw_qpolynomial_fold_cow` (+1 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1254-1283

````c
__isl_give isl_pw_qpolynomial_fold *isl_pw_qpolynomial_fold_from_pw_qpolynomial(
	enum isl_fold type, __isl_take isl_pw_qpolynomial *pwqp)
{
	int i;
	isl_pw_qpolynomial_fold *pwf;

	if (!pwqp)
		return NULL;
	
	pwf = isl_pw_qpolynomial_fold_alloc_size(isl_space_copy(pwqp->dim),
						type, pwqp->n);

	for (i = 0; i < pwqp->n; ++i)
		pwf = isl_pw_qpolynomial_fold_add_piece(pwf,
			isl_set_copy(pwqp->p[i].set),
			isl_qpolynomial_fold_alloc(type,
				isl_qpolynomial_copy(pwqp->p[i].qp)));

	isl_pw_qpolynomial_free(pwqp);

	return pwf;
}

__isl_give isl_pw_qpolynomial_fold *isl_pw_qpolynomial_fold_add(
	__isl_take isl_pw_qpolynomial_fold *pwf1,
	__isl_take isl_pw_qpolynomial_fold *pwf2)
{
	return isl_pw_qpolynomial_fold_union_add_(pwf1, pwf2);
}

````
- **EN**: This block defines enum values such as `isl_fold`; declares or defines routines around `isl_pw_qpolynomial_fold_from_pw_qpolynomial`, `isl_pw_qpolynomial_fold_alloc_size`, `isl_pw_qpolynomial_fold_add_piece`, `isl_set_copy` (+4 more); contains control flow with 1 loop construct(s), 1 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 定义枚举类型，例如 `isl_fold`; 声明或定义与 `isl_pw_qpolynomial_fold_from_pw_qpolynomial`, `isl_pw_qpolynomial_fold_alloc_size`, `isl_pw_qpolynomial_fold_add_piece`, `isl_set_copy` (+4 more) 相关的例程; 包含控制流结构：1 处循环、1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 1284-1309

````c
/* Compare two quasi-polynomial reductions.
 *
 * Return -1 if "fold1" is "smaller" than "fold2", 1 if "fold1" is "greater"
 * than "fold2" and 0 if they are equal.
 */
int isl_qpolynomial_fold_plain_cmp(__isl_keep isl_qpolynomial_fold *fold1,
	__isl_keep isl_qpolynomial_fold *fold2)
{
	int i;
	isl_size n1, n2;
	isl_qpolynomial_list *list1, *list2;

	if (fold1 == fold2)
		return 0;
	list1 = isl_qpolynomial_fold_peek_list(fold1);
	list2 = isl_qpolynomial_fold_peek_list(fold2);
	n1 = isl_qpolynomial_list_size(list1);
	n2 = isl_qpolynomial_list_size(list2);
	if (n1 < 0)
		return -1;
	if (n2 < 0)
		return 1;

	if (n1 != n2)
		return n1 - n2;

````
- **EN**: This block declares or defines routines around `isl_qpolynomial_fold_plain_cmp`, `isl_qpolynomial_fold_peek_list`, `isl_qpolynomial_list_size`; contains control flow with 4 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_qpolynomial_fold_plain_cmp`, `isl_qpolynomial_fold_peek_list`, `isl_qpolynomial_list_size` 相关的例程; 包含控制流结构：4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 1310-1335

````c
	for (i = 0; i < n1; ++i) {
		int cmp;
		isl_qpolynomial *qp1, *qp2;

		qp1 = isl_qpolynomial_list_peek(list1, i);
		qp2 = isl_qpolynomial_list_peek(list2, i);
		cmp = isl_qpolynomial_plain_cmp(qp1, qp2);
		if (cmp != 0)
			return cmp;
	}

	return 0;
}

/* Are the lists "list1" and "list2", both consisting of "n" elements
 * obviously equal to each other?
 */
static isl_bool isl_qpolynomial_list_plain_is_equal(unsigned n,
	isl_qpolynomial_list *list1, isl_qpolynomial_list *list2)
{
	int i;

	for (i = 0; i < n; ++i) {
		isl_bool eq;
		isl_qpolynomial *qp1, *qp2;

````
- **EN**: This block declares or defines routines around `isl_qpolynomial_list_peek`, `isl_qpolynomial_plain_cmp`, `isl_qpolynomial_list_plain_is_equal`; contains control flow with 2 loop construct(s), 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_qpolynomial_list_peek`, `isl_qpolynomial_plain_cmp`, `isl_qpolynomial_list_plain_is_equal` 相关的例程; 包含控制流结构：2 处循环、1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 1336-1362

````c
		qp1 = isl_qpolynomial_list_peek(list1, i);
		qp2 = isl_qpolynomial_list_peek(list2, i);
		eq = isl_qpolynomial_plain_is_equal(qp1, qp2);
		if (eq < 0 || !eq)
			return eq;
	}

	return isl_bool_true;
}

/* Wrapper around isl_qpolynomial_plain_cmp for use
 * as a isl_qpolynomial_list_sort callback.
 */
static int qpolynomial_cmp(__isl_keep isl_qpolynomial *a,
	__isl_keep isl_qpolynomial *b, void *user)
{
	return isl_qpolynomial_plain_cmp(a, b);
}

isl_bool isl_qpolynomial_fold_plain_is_equal(
	__isl_keep isl_qpolynomial_fold *fold1,
	__isl_keep isl_qpolynomial_fold *fold2)
{
	isl_bool equal;
	isl_size n1, n2;
	isl_qpolynomial_list *list1, *list2;

````
- **EN**: This block declares or defines routines around `isl_qpolynomial_list_peek`, `isl_qpolynomial_plain_is_equal`, `qpolynomial_cmp`, `isl_qpolynomial_fold_plain_is_equal`; contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_qpolynomial_list_peek`, `isl_qpolynomial_plain_is_equal`, `qpolynomial_cmp`, `isl_qpolynomial_fold_plain_is_equal` 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 1363-1391

````c
	list1 = isl_qpolynomial_fold_peek_list(fold1);
	list2 = isl_qpolynomial_fold_peek_list(fold2);
	n1 = isl_qpolynomial_list_size(list1);
	n2 = isl_qpolynomial_list_size(list2);
	if (n1 < 0 || n2 < 0)
		return isl_bool_error;

	if (n1 != n2)
		return isl_bool_false;

	list1 = isl_qpolynomial_list_copy(list1);
	list1 = isl_qpolynomial_list_sort(list1, &qpolynomial_cmp, NULL);
	list2 = isl_qpolynomial_list_copy(list2);
	list2 = isl_qpolynomial_list_sort(list2, &qpolynomial_cmp, NULL);
	equal = isl_qpolynomial_list_plain_is_equal(n1, list1, list2);
	isl_qpolynomial_list_free(list1);
	isl_qpolynomial_list_free(list2);
	return equal;
}

__isl_give isl_val *isl_qpolynomial_fold_eval(
	__isl_take isl_qpolynomial_fold *fold, __isl_take isl_point *pnt)
{
	isl_size n;
	isl_ctx *ctx;
	isl_val *v;
	isl_qpolynomial *qp;
	isl_qpolynomial_list *list;

````
- **EN**: This block declares or defines routines around `isl_qpolynomial_fold_peek_list`, `isl_qpolynomial_list_size`, `isl_qpolynomial_list_copy`, `isl_qpolynomial_list_sort` (+3 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_qpolynomial_fold_peek_list`, `isl_qpolynomial_list_size`, `isl_qpolynomial_list_copy`, `isl_qpolynomial_list_sort` (+3 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1392-1425

````c
	if (!fold || !pnt)
		goto error;
	ctx = isl_point_get_ctx(pnt);
	isl_assert(pnt->dim->ctx, isl_space_is_equal(pnt->dim, fold->dim), goto error);
	isl_assert(pnt->dim->ctx,
		fold->type == isl_fold_max || fold->type == isl_fold_min,
		goto error);

	list = isl_qpolynomial_fold_peek_list(fold);
	n = isl_qpolynomial_list_size(list);
	if (n < 0)
		goto error;

	if (n == 0)
		v = isl_val_zero(ctx);
	else {
		int i;

		qp = isl_qpolynomial_list_get_at(list, 0);
		v = isl_qpolynomial_eval(qp, isl_point_copy(pnt));
		for (i = 1; i < n; ++i) {
			isl_val *v_i;

			qp = isl_qpolynomial_list_get_at(list, i);
			v_i = isl_qpolynomial_eval(qp, isl_point_copy(pnt));
			if (fold->type == isl_fold_max)
				v = isl_val_max(v, v_i);
			else
				v = isl_val_min(v, v_i);
		}
	}
	isl_qpolynomial_fold_free(fold);
	isl_point_free(pnt);

````
- **EN**: This block declares or defines routines around `isl_point_get_ctx`, `isl_assert`, `isl_qpolynomial_fold_peek_list`, `isl_qpolynomial_list_size` (+7 more); contains control flow with 1 loop construct(s), 4 conditional check(s); adds defensive checks for invariants or impossible states; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_point_get_ctx`, `isl_assert`, `isl_qpolynomial_fold_peek_list`, `isl_qpolynomial_list_size` (+7 more) 相关的例程; 包含控制流结构：1 处循环、4 处条件判断; 加入用于保护不变量或异常状态的断言/检查; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1426-1452

````c
	return v;
error:
	isl_qpolynomial_fold_free(fold);
	isl_point_free(pnt);
	return NULL;
}

size_t isl_pw_qpolynomial_fold_size(__isl_keep isl_pw_qpolynomial_fold *pwf)
{
	int i;
	size_t n = 0;

	for (i = 0; i < pwf->n; ++i) {
		isl_size n_i;
		isl_qpolynomial_list *list;

		list = isl_qpolynomial_fold_peek_list(pwf->p[i].fold);
		n_i = isl_qpolynomial_list_size(list);
		if (n_i < 0)
			return isl_size_error;

		n += n_i;
	}

	return n;
}

````
- **EN**: This block declares or defines routines around `isl_qpolynomial_fold_free`, `isl_point_free`, `isl_pw_qpolynomial_fold_size`, `isl_qpolynomial_fold_peek_list` (+1 more); contains control flow with 1 loop construct(s), 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_qpolynomial_fold_free`, `isl_point_free`, `isl_pw_qpolynomial_fold_size`, `isl_qpolynomial_fold_peek_list` (+1 more) 相关的例程; 包含控制流结构：1 处循环、1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1453-1478

````c
__isl_give isl_val *isl_qpolynomial_fold_opt_on_domain(
	__isl_take isl_qpolynomial_fold *fold, __isl_take isl_set *set, int max)
{
	int i;
	isl_size n;
	isl_val *opt;
	isl_qpolynomial *qp;
	isl_qpolynomial_list *list;

	list = isl_qpolynomial_fold_peek_list(fold);
	n = isl_qpolynomial_list_size(list);
	if (!set || n < 0)
		goto error;

	if (n == 0) {
		opt = isl_val_zero(isl_set_get_ctx(set));
		isl_set_free(set);
		isl_qpolynomial_fold_free(fold);
		return opt;
	}

	qp = isl_qpolynomial_list_get_at(list, 0);
	opt = isl_qpolynomial_opt_on_domain(qp, isl_set_copy(set), max);
	for (i = 1; i < n; ++i) {
		isl_val *opt_i;

````
- **EN**: This block declares or defines routines around `isl_qpolynomial_fold_opt_on_domain`, `isl_qpolynomial_fold_peek_list`, `isl_qpolynomial_list_size`, `isl_val_zero` (+4 more); contains control flow with 1 loop construct(s), 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_qpolynomial_fold_opt_on_domain`, `isl_qpolynomial_fold_peek_list`, `isl_qpolynomial_list_size`, `isl_val_zero` (+4 more) 相关的例程; 包含控制流结构：1 处循环、2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1479-1509

````c
		qp = isl_qpolynomial_list_get_at(list, i);
		opt_i = isl_qpolynomial_opt_on_domain(qp,
				isl_set_copy(set), max);
		if (max)
			opt = isl_val_max(opt, opt_i);
		else
			opt = isl_val_min(opt, opt_i);
	}

	isl_set_free(set);
	isl_qpolynomial_fold_free(fold);

	return opt;
error:
	isl_set_free(set);
	isl_qpolynomial_fold_free(fold);
	return NULL;
}

/* Check whether for each quasi-polynomial in "fold2" there is
 * a quasi-polynomial in "fold1" that dominates it on "set".
 */
static isl_bool qpolynomial_fold_covers_on_domain(__isl_keep isl_set *set,
	__isl_keep isl_qpolynomial_fold *fold1,
	__isl_keep isl_qpolynomial_fold *fold2)
{
	int i, j;
	int covers;
	isl_size n1, n2;
	isl_qpolynomial_list *list1, *list2;

````
- **EN**: This block declares or defines routines around `isl_qpolynomial_list_get_at`, `isl_qpolynomial_opt_on_domain`, `isl_set_copy`, `isl_val_max` (+4 more); contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_qpolynomial_list_get_at`, `isl_qpolynomial_opt_on_domain`, `isl_set_copy`, `isl_val_max` (+4 more) 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 1510-1535

````c
	list1 = isl_qpolynomial_fold_peek_list(fold1);
	list2 = isl_qpolynomial_fold_peek_list(fold2);
	n1 = isl_qpolynomial_list_size(list1);
	n2 = isl_qpolynomial_list_size(list2);
	if (!set || n1 < 0 || n2 < 0)
		return isl_bool_error;

	covers = fold1->type == isl_fold_max ? 1 : -1;

	for (i = 0; i < n2; ++i) {
		for (j = 0; j < n1; ++j) {
			isl_qpolynomial *qp1, *qp2, *d;
			int sgn;

			qp1 = isl_qpolynomial_list_get_at(list1, j);
			qp2 = isl_qpolynomial_list_get_at(list2, i);
			d = isl_qpolynomial_sub(qp1, qp2);
			sgn = isl_qpolynomial_sign(set, d);
			isl_qpolynomial_free(d);
			if (sgn == covers)
				break;
		}
		if (j >= n1)
			return isl_bool_false;
	}

````
- **EN**: This block declares or defines routines around `isl_qpolynomial_fold_peek_list`, `isl_qpolynomial_list_size`, `isl_qpolynomial_list_get_at`, `isl_qpolynomial_sub` (+2 more); contains control flow with 2 loop construct(s), 3 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_qpolynomial_fold_peek_list`, `isl_qpolynomial_list_size`, `isl_qpolynomial_list_get_at`, `isl_qpolynomial_sub` (+2 more) 相关的例程; 包含控制流结构：2 处循环、3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1536-1564

````c
	return isl_bool_true;
}

/* Check whether "pwf1" dominated "pwf2", i.e., the domain of "pwf1" contains
 * that of "pwf2" and on each cell, the corresponding fold from pwf1 dominates
 * that of pwf2.
 */
isl_bool isl_pw_qpolynomial_fold_covers(
	__isl_keep isl_pw_qpolynomial_fold *pwf1,
	__isl_keep isl_pw_qpolynomial_fold *pwf2)
{
	int i, j;
	isl_set *dom1, *dom2;
	isl_bool is_subset;

	if (!pwf1 || !pwf2)
		return isl_bool_error;

	if (pwf2->n == 0)
		return isl_bool_true;
	if (pwf1->n == 0)
		return isl_bool_false;

	dom1 = isl_pw_qpolynomial_fold_domain(isl_pw_qpolynomial_fold_copy(pwf1));
	dom2 = isl_pw_qpolynomial_fold_domain(isl_pw_qpolynomial_fold_copy(pwf2));
	is_subset = isl_set_is_subset(dom2, dom1);
	isl_set_free(dom1);
	isl_set_free(dom2);

````
- **EN**: This block declares or defines routines around `isl_pw_qpolynomial_fold_covers`, `isl_pw_qpolynomial_fold_domain`, `isl_set_is_subset`, `isl_set_free`; contains control flow with 3 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_pw_qpolynomial_fold_covers`, `isl_pw_qpolynomial_fold_domain`, `isl_set_is_subset`, `isl_set_free` 相关的例程; 包含控制流结构：3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 1565-1590

````c
	if (is_subset < 0 || !is_subset)
		return is_subset;

	for (i = 0; i < pwf2->n; ++i) {
		for (j = 0; j < pwf1->n; ++j) {
			isl_bool is_empty;
			isl_set *common;
			isl_bool covers;

			common = isl_set_intersect(isl_set_copy(pwf1->p[j].set),
						   isl_set_copy(pwf2->p[i].set));
			is_empty = isl_set_is_empty(common);
			if (is_empty < 0 || is_empty) {
				isl_set_free(common);
				if (is_empty < 0)
					return isl_bool_error;
				continue;
			}
			covers = qpolynomial_fold_covers_on_domain(common,
					pwf1->p[j].fold, pwf2->p[i].fold);
			isl_set_free(common);
			if (covers < 0 || !covers)
				return covers;
		}
	}

````
- **EN**: This block declares or defines routines around `isl_set_intersect`, `isl_set_copy`, `isl_set_is_empty`, `isl_set_free` (+1 more); contains control flow with 2 loop construct(s), 4 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_set_intersect`, `isl_set_copy`, `isl_set_is_empty`, `isl_set_free` (+1 more) 相关的例程; 包含控制流结构：2 处循环、4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1591-1618

````c
	return isl_bool_true;
}

/* isl_qpolynomial_list_map callback that calls
 * isl_qpolynomial_morph_domain on "qp".
 */
static __isl_give isl_qpolynomial *morph_domain(
	__isl_take isl_qpolynomial *qp, void *user)
{
	isl_morph *morph = user;

	return isl_qpolynomial_morph_domain(qp, isl_morph_copy(morph));
}

__isl_give isl_qpolynomial_fold *isl_qpolynomial_fold_morph_domain(
	__isl_take isl_qpolynomial_fold *fold, __isl_take isl_morph *morph)
{
	isl_space *space;
	isl_qpolynomial_list *list;

	space = isl_qpolynomial_fold_peek_domain_space(fold);
	if (isl_morph_check_applies(morph, space) < 0)
		goto error;

	list = isl_qpolynomial_fold_take_list(fold);
	list = isl_qpolynomial_list_map(list, &morph_domain, morph);
	fold = isl_qpolynomial_fold_restore_list(fold, list);

````
- **EN**: This block declares or defines routines around `morph_domain`, `isl_qpolynomial_fold_morph_domain`, `isl_qpolynomial_fold_peek_domain_space`, `isl_qpolynomial_fold_take_list` (+2 more); contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `morph_domain`, `isl_qpolynomial_fold_morph_domain`, `isl_qpolynomial_fold_peek_domain_space`, `isl_qpolynomial_fold_take_list` (+2 more) 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 1619-1648

````c
	space = isl_morph_get_ran_space(morph);
	isl_space_free(isl_qpolynomial_fold_take_domain_space(fold));
	fold = isl_qpolynomial_fold_restore_domain_space(fold, space);

	isl_morph_free(morph);

	return fold;
error:
	isl_qpolynomial_fold_free(fold);
	isl_morph_free(morph);
	return NULL;
}

enum isl_fold isl_qpolynomial_fold_get_type(__isl_keep isl_qpolynomial_fold *fold)
{
	if (!fold)
		return isl_fold_error;
	return fold->type;
}

/* Return the type of this piecewise quasipolynomial reduction.
 */
enum isl_fold isl_pw_qpolynomial_fold_get_type(
	__isl_keep isl_pw_qpolynomial_fold *pwf)
{
	if (!pwf)
		return isl_fold_error;
	return pwf->type;
}

````
- **EN**: This block defines enum values such as `isl_fold`; declares or defines routines around `isl_morph_get_ran_space`, `isl_space_free`, `isl_qpolynomial_fold_restore_domain_space`, `isl_morph_free` (+3 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 定义枚举类型，例如 `isl_fold`; 声明或定义与 `isl_morph_get_ran_space`, `isl_space_free`, `isl_qpolynomial_fold_restore_domain_space`, `isl_morph_free` (+3 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 1649-1675

````c
enum isl_fold isl_union_pw_qpolynomial_fold_get_type(
	__isl_keep isl_union_pw_qpolynomial_fold *upwf)
{
	if (!upwf)
		return isl_fold_error;
	return upwf->type;
}

/* isl_qpolynomial_list_map callback that calls
 * isl_qpolynomial_lift on "qp".
 */
static __isl_give isl_qpolynomial *lift(__isl_take isl_qpolynomial *qp,
	void *user)
{
	isl_space *space = user;

	return isl_qpolynomial_lift(qp, isl_space_copy(space));
}

__isl_give isl_qpolynomial_fold *isl_qpolynomial_fold_lift(
	__isl_take isl_qpolynomial_fold *fold, __isl_take isl_space *space)
{
	isl_qpolynomial_list *list;

	if (!fold || !space)
		goto error;

````
- **EN**: This block defines enum values such as `isl_fold`; declares or defines routines around `isl_union_pw_qpolynomial_fold_get_type`, `lift`, `isl_qpolynomial_fold_lift`; contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 定义枚举类型，例如 `isl_fold`; 声明或定义与 `isl_union_pw_qpolynomial_fold_get_type`, `lift`, `isl_qpolynomial_fold_lift` 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 1676-1704

````c
	if (isl_space_is_equal(fold->dim, space)) {
		isl_space_free(space);
		return fold;
	}

	list = isl_qpolynomial_fold_take_list(fold);
	list = isl_qpolynomial_list_map(list, &lift, space);
	fold = isl_qpolynomial_fold_restore_list(fold, list);

	isl_space_free(isl_qpolynomial_fold_take_domain_space(fold));
	fold = isl_qpolynomial_fold_restore_domain_space(fold, space);

	return fold;
error:
	isl_qpolynomial_fold_free(fold);
	isl_space_free(space);
	return NULL;
}

isl_stat isl_qpolynomial_fold_foreach_qpolynomial(
	__isl_keep isl_qpolynomial_fold *fold,
	isl_stat (*fn)(__isl_take isl_qpolynomial *qp, void *user), void *user)
{
	isl_qpolynomial_list *list;

	list = isl_qpolynomial_fold_peek_list(fold);
	return isl_qpolynomial_list_foreach(list, fn, user);
}

````
- **EN**: This block declares or defines routines around `isl_space_free`, `isl_qpolynomial_fold_take_list`, `isl_qpolynomial_list_map`, `isl_qpolynomial_fold_restore_list` (+5 more); contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_space_free`, `isl_qpolynomial_fold_take_list`, `isl_qpolynomial_list_map`, `isl_qpolynomial_fold_restore_list` (+5 more) 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1705-1739

````c
/* Internal data structure for isl_qpolynomial_fold_move_dims
 * representing its arguments.
 */
struct isl_fold_move_dims_data {
	enum isl_dim_type dst_type;
	unsigned dst_pos;
	enum isl_dim_type src_type;
	unsigned src_pos;
	unsigned n;
};

/* isl_qpolynomial_list_map callback for calling
 * isl_qpolynomial_move_dims on "qp".
 */
static __isl_give isl_qpolynomial *move_dims(__isl_take isl_qpolynomial *qp,
	void *user)
{
	struct isl_fold_move_dims_data *data = user;

	qp = isl_qpolynomial_move_dims(qp, data->dst_type, data->dst_pos,
					data->src_type, data->src_pos, data->n);
	return qp;
}

__isl_give isl_qpolynomial_fold *isl_qpolynomial_fold_move_dims(
	__isl_take isl_qpolynomial_fold *fold,
	enum isl_dim_type dst_type, unsigned dst_pos,
	enum isl_dim_type src_type, unsigned src_pos, unsigned n)
{
	struct isl_fold_move_dims_data data =
		{ dst_type, dst_pos, src_type, src_pos, n };
	enum isl_dim_type set_src_type, set_dst_type;
	isl_space *space;
	isl_qpolynomial_list *list;

````
- **EN**: This block declares or references types such as `isl_fold_move_dims_data`; defines enum values such as `isl_dim_type`; declares or defines routines around `move_dims`, `isl_qpolynomial_move_dims`, `isl_qpolynomial_fold_move_dims`; emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_fold_move_dims_data`; 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `move_dims`, `isl_qpolynomial_move_dims`, `isl_qpolynomial_fold_move_dims` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 1740-1771

````c
	if (n == 0)
		return fold;

	fold = isl_qpolynomial_fold_cow(fold);
	if (!fold)
		return NULL;

	set_src_type = domain_type(src_type);
	set_dst_type = domain_type(dst_type);

	list = isl_qpolynomial_fold_take_list(fold);
	list = isl_qpolynomial_list_map(list, &move_dims, &data);
	fold = isl_qpolynomial_fold_restore_list(fold, list);

	space = isl_qpolynomial_fold_take_domain_space(fold);
	space = isl_space_move_dims(space, set_dst_type, dst_pos,
						set_src_type, src_pos, n);
	fold = isl_qpolynomial_fold_restore_domain_space(fold, space);

	return fold;
}

/* Internal data structure for isl_qpolynomial_fold_substitute
 * representing its arguments.
 */
struct isl_fold_substitute {
	enum isl_dim_type type;
	unsigned first;
	unsigned n;
	isl_qpolynomial **subs;
};

````
- **EN**: This block declares or references types such as `isl_fold_substitute`; defines enum values such as `isl_dim_type`; declares or defines routines around `isl_qpolynomial_fold_cow`, `domain_type`, `isl_qpolynomial_fold_take_list`, `isl_qpolynomial_list_map` (+4 more); contains control flow with 2 conditional check(s); and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_fold_substitute`; 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_qpolynomial_fold_cow`, `domain_type`, `isl_qpolynomial_fold_take_list`, `isl_qpolynomial_list_map` (+4 more) 相关的例程; 包含控制流结构：2 处条件判断；并延续周边实现细节。

### Lines 1772-1798

````c
/* isl_qpolynomial_list_map callback for calling
 * isl_qpolynomial_substitute on "qp".
 */
static __isl_give isl_qpolynomial *substitute(__isl_take isl_qpolynomial *qp,
	void *user)
{
	struct isl_fold_substitute *data = user;

	qp = isl_qpolynomial_substitute(qp,
				data->type, data->first, data->n, data->subs);
	return qp;
}

/* For each 0 <= i < "n", replace variable "first" + i of type "type"
 * in fold->qp[k] by subs[i].
 */
__isl_give isl_qpolynomial_fold *isl_qpolynomial_fold_substitute(
	__isl_take isl_qpolynomial_fold *fold,
	enum isl_dim_type type, unsigned first, unsigned n,
	__isl_keep isl_qpolynomial **subs)
{
	struct isl_fold_substitute data = { type, first, n, subs };
	isl_qpolynomial_list *list;

	if (n == 0)
		return fold;

````
- **EN**: This block declares or references types such as `isl_fold_substitute`; defines enum values such as `isl_dim_type`; declares or defines routines around `substitute`, `isl_qpolynomial_substitute`, `isl_qpolynomial_fold_substitute`; contains control flow with 1 conditional check(s); and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_fold_substitute`; 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `substitute`, `isl_qpolynomial_substitute`, `isl_qpolynomial_fold_substitute` 相关的例程; 包含控制流结构：1 处条件判断；并延续周边实现细节。

### Lines 1799-1830

````c
	list = isl_qpolynomial_fold_take_list(fold);
	list = isl_qpolynomial_list_map(list, &substitute, &data);
	fold = isl_qpolynomial_fold_restore_list(fold, list);

	return fold;
}

static isl_stat add_pwqp(__isl_take isl_pw_qpolynomial *pwqp, void *user)
{
	isl_pw_qpolynomial_fold *pwf;
	isl_union_pw_qpolynomial_fold **upwf;
	struct isl_hash_table_entry *entry;

	upwf = (isl_union_pw_qpolynomial_fold **)user;

	entry = isl_union_pw_qpolynomial_fold_find_part_entry(*upwf,
			 pwqp->dim, 1);
	if (!entry)
		goto error;

	pwf = isl_pw_qpolynomial_fold_from_pw_qpolynomial((*upwf)->type, pwqp);
	if (!entry->data)
		entry->data = pwf;
	else {
		entry->data = isl_pw_qpolynomial_fold_add(entry->data, pwf);
		if (!entry->data)
			return isl_stat_error;
		if (isl_pw_qpolynomial_fold_is_zero(entry->data))
			*upwf = isl_union_pw_qpolynomial_fold_remove_part_entry(
								*upwf, entry);
	}

````
- **EN**: This block declares or references types such as `isl_hash_table_entry`; declares or defines routines around `isl_qpolynomial_fold_take_list`, `isl_qpolynomial_list_map`, `isl_qpolynomial_fold_restore_list`, `add_pwqp` (+4 more); contains control flow with 4 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_hash_table_entry`; 声明或定义与 `isl_qpolynomial_fold_take_list`, `isl_qpolynomial_list_map`, `isl_qpolynomial_fold_restore_list`, `add_pwqp` (+4 more) 相关的例程; 包含控制流结构：4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 1831-1862

````c
	return isl_stat_ok;
error:
	isl_pw_qpolynomial_free(pwqp);
	return isl_stat_error;
}

__isl_give isl_union_pw_qpolynomial_fold *isl_union_pw_qpolynomial_fold_add_union_pw_qpolynomial(
	__isl_take isl_union_pw_qpolynomial_fold *upwf,
	__isl_take isl_union_pw_qpolynomial *upwqp)
{
	upwf = isl_union_pw_qpolynomial_fold_align_params(upwf,
				isl_union_pw_qpolynomial_get_space(upwqp));
	upwqp = isl_union_pw_qpolynomial_align_params(upwqp,
				isl_union_pw_qpolynomial_fold_get_space(upwf));

	upwf = isl_union_pw_qpolynomial_fold_cow(upwf);
	if (!upwf || !upwqp)
		goto error;

	if (isl_union_pw_qpolynomial_foreach_pw_qpolynomial(upwqp, &add_pwqp,
							 &upwf) < 0)
		goto error;

	isl_union_pw_qpolynomial_free(upwqp);

	return upwf;
error:
	isl_union_pw_qpolynomial_fold_free(upwf);
	isl_union_pw_qpolynomial_free(upwqp);
	return NULL;
}

````
- **EN**: This block declares or defines routines around `isl_pw_qpolynomial_free`, `isl_union_pw_qpolynomial_fold_add_union_pw_qpolynomial`, `isl_union_pw_qpolynomial_fold_align_params`, `isl_union_pw_qpolynomial_get_space` (+5 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_pw_qpolynomial_free`, `isl_union_pw_qpolynomial_fold_add_union_pw_qpolynomial`, `isl_union_pw_qpolynomial_fold_align_params`, `isl_union_pw_qpolynomial_get_space` (+5 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1863-1893

````c
static isl_bool join_compatible(__isl_keep isl_space *space1,
	__isl_keep isl_space *space2)
{
	isl_bool m;
	m = isl_space_has_equal_params(space1, space2);
	if (m < 0 || !m)
		return m;
	return isl_space_tuple_is_equal(space1, isl_dim_out,
					space2, isl_dim_in);
}

/* Compute the intersection of the range of the map and the domain
 * of the piecewise quasipolynomial reduction and then compute a bound
 * on the associated quasipolynomial reduction over all elements
 * in this intersection.
 *
 * We first introduce some unconstrained dimensions in the
 * piecewise quasipolynomial, intersect the resulting domain
 * with the wrapped map and the compute the sum.
 */
__isl_give isl_pw_qpolynomial_fold *isl_map_apply_pw_qpolynomial_fold(
	__isl_take isl_map *map, __isl_take isl_pw_qpolynomial_fold *pwf,
	isl_bool *tight)
{
	isl_ctx *ctx;
	isl_set *dom;
	isl_space *map_space;
	isl_space *pwf_space;
	isl_size n_in;
	isl_bool ok;

````
- **EN**: This block declares or defines routines around `join_compatible`, `isl_space_has_equal_params`, `isl_map_apply_pw_qpolynomial_fold`; contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `join_compatible`, `isl_space_has_equal_params`, `isl_map_apply_pw_qpolynomial_fold` 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 1894-1920

````c
	ctx = isl_map_get_ctx(map);
	if (!ctx)
		goto error;

	map_space = isl_map_get_space(map);
	pwf_space = isl_pw_qpolynomial_fold_get_space(pwf);
	ok = join_compatible(map_space, pwf_space);
	isl_space_free(map_space);
	isl_space_free(pwf_space);
	if (ok < 0)
		goto error;
	if (!ok)
		isl_die(ctx, isl_error_invalid, "incompatible dimensions",
			goto error);

	n_in = isl_map_dim(map, isl_dim_in);
	if (n_in < 0)
		goto error;
	pwf = isl_pw_qpolynomial_fold_insert_dims(pwf, isl_dim_in, 0, n_in);

	dom = isl_map_wrap(map);
	pwf = isl_pw_qpolynomial_fold_reset_domain_space(pwf,
						isl_set_get_space(dom));

	pwf = isl_pw_qpolynomial_fold_intersect_domain(pwf, dom);
	pwf = isl_pw_qpolynomial_fold_bound(pwf, tight);
	
````
- **EN**: This block declares or defines routines around `isl_map_get_ctx`, `isl_map_get_space`, `isl_pw_qpolynomial_fold_get_space`, `join_compatible` (+9 more); contains control flow with 4 conditional check(s); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_map_get_ctx`, `isl_map_get_space`, `isl_pw_qpolynomial_fold_get_space`, `join_compatible` (+9 more) 相关的例程; 包含控制流结构：4 处条件判断; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1921-1949

````c
	return pwf;
error:
	isl_map_free(map);
	isl_pw_qpolynomial_fold_free(pwf);
	return NULL;
}

__isl_give isl_pw_qpolynomial_fold *isl_set_apply_pw_qpolynomial_fold(
	__isl_take isl_set *set, __isl_take isl_pw_qpolynomial_fold *pwf,
	isl_bool *tight)
{
	return isl_map_apply_pw_qpolynomial_fold(set, pwf, tight);
}

struct isl_apply_fold_data {
	isl_union_pw_qpolynomial_fold *upwf;
	isl_union_pw_qpolynomial_fold *res;
	isl_map *map;
	isl_bool tight;
};

static isl_stat pw_qpolynomial_fold_apply(
	__isl_take isl_pw_qpolynomial_fold *pwf, void *user)
{
	isl_space *map_dim;
	isl_space *pwf_dim;
	struct isl_apply_fold_data *data = user;
	isl_bool ok;

````
- **EN**: This block declares or references types such as `isl_apply_fold_data`; declares or defines routines around `isl_map_free`, `isl_pw_qpolynomial_fold_free`, `isl_set_apply_pw_qpolynomial_fold`, `pw_qpolynomial_fold_apply`; emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或引用类型，例如 `isl_apply_fold_data`; 声明或定义与 `isl_map_free`, `isl_pw_qpolynomial_fold_free`, `isl_set_apply_pw_qpolynomial_fold`, `pw_qpolynomial_fold_apply` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1950-1977

````c
	map_dim = isl_map_get_space(data->map);
	pwf_dim = isl_pw_qpolynomial_fold_get_space(pwf);
	ok = join_compatible(map_dim, pwf_dim);
	isl_space_free(map_dim);
	isl_space_free(pwf_dim);

	if (ok < 0)
		return isl_stat_error;
	if (ok) {
		pwf = isl_map_apply_pw_qpolynomial_fold(isl_map_copy(data->map),
				    pwf, data->tight ? &data->tight : NULL);
		data->res = isl_union_pw_qpolynomial_fold_fold_pw_qpolynomial_fold(
							data->res, pwf);
	} else
		isl_pw_qpolynomial_fold_free(pwf);

	return isl_stat_ok;
}

static isl_stat map_apply(__isl_take isl_map *map, void *user)
{
	struct isl_apply_fold_data *data = user;
	isl_stat r;

	data->map = map;
	r = isl_union_pw_qpolynomial_fold_foreach_pw_qpolynomial_fold(
				data->upwf, &pw_qpolynomial_fold_apply, data);

````
- **EN**: This block declares or references types such as `isl_apply_fold_data`; declares or defines routines around `isl_map_get_space`, `isl_pw_qpolynomial_fold_get_space`, `join_compatible`, `isl_space_free` (+5 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_apply_fold_data`; 声明或定义与 `isl_map_get_space`, `isl_pw_qpolynomial_fold_get_space`, `join_compatible`, `isl_space_free` (+5 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 1978-2005

````c
	isl_map_free(map);
	return r;
}

__isl_give isl_union_pw_qpolynomial_fold *isl_union_map_apply_union_pw_qpolynomial_fold(
	__isl_take isl_union_map *umap,
	__isl_take isl_union_pw_qpolynomial_fold *upwf, isl_bool *tight)
{
	isl_space *space;
	enum isl_fold type;
	struct isl_apply_fold_data data;

	upwf = isl_union_pw_qpolynomial_fold_align_params(upwf,
				isl_union_map_get_space(umap));
	umap = isl_union_map_align_params(umap,
				isl_union_pw_qpolynomial_fold_get_space(upwf));

	data.upwf = upwf;
	data.tight = tight ? isl_bool_true : isl_bool_false;
	space = isl_union_pw_qpolynomial_fold_get_space(upwf);
	type = isl_union_pw_qpolynomial_fold_get_type(upwf);
	data.res = isl_union_pw_qpolynomial_fold_zero(space, type);
	if (isl_union_map_foreach_map(umap, &map_apply, &data) < 0)
		goto error;

	isl_union_map_free(umap);
	isl_union_pw_qpolynomial_fold_free(upwf);

````
- **EN**: This block declares or references types such as `isl_apply_fold_data`; defines enum values such as `isl_fold`; declares or defines routines around `isl_map_free`, `isl_union_map_apply_union_pw_qpolynomial_fold`, `isl_union_pw_qpolynomial_fold_align_params`, `isl_union_map_get_space` (+6 more); contains control flow with 1 conditional check(s); and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_apply_fold_data`; 定义枚举类型，例如 `isl_fold`; 声明或定义与 `isl_map_free`, `isl_union_map_apply_union_pw_qpolynomial_fold`, `isl_union_pw_qpolynomial_fold_align_params`, `isl_union_map_get_space` (+6 more) 相关的例程; 包含控制流结构：1 处条件判断；并延续周边实现细节。

### Lines 2006-2031

````c
	if (tight)
		*tight = data.tight;

	return data.res;
error:
	isl_union_map_free(umap);
	isl_union_pw_qpolynomial_fold_free(upwf);
	isl_union_pw_qpolynomial_fold_free(data.res);
	return NULL;
}

__isl_give isl_union_pw_qpolynomial_fold *isl_union_set_apply_union_pw_qpolynomial_fold(
	__isl_take isl_union_set *uset,
	__isl_take isl_union_pw_qpolynomial_fold *upwf, isl_bool *tight)
{
	return isl_union_map_apply_union_pw_qpolynomial_fold(uset, upwf, tight);
}

/* isl_qpolynomial_list_map callback for calling
 * isl_qpolynomial_realign_domain on "qp".
 */
static __isl_give isl_qpolynomial *realign_domain(
	__isl_take isl_qpolynomial *qp, void *user)
{
	isl_reordering *r = user;

````
- **EN**: This block declares or defines routines around `isl_union_map_free`, `isl_union_pw_qpolynomial_fold_free`, `isl_union_set_apply_union_pw_qpolynomial_fold`, `realign_domain`; contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_union_map_free`, `isl_union_pw_qpolynomial_fold_free`, `isl_union_set_apply_union_pw_qpolynomial_fold`, `realign_domain` 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 2032-2063

````c
	qp = isl_qpolynomial_realign_domain(qp, isl_reordering_copy(r));
	return qp;
}

/* Reorder the dimension of "fold" according to the given reordering.
 */
__isl_give isl_qpolynomial_fold *isl_qpolynomial_fold_realign_domain(
	__isl_take isl_qpolynomial_fold *fold, __isl_take isl_reordering *r)
{
	isl_space *space;
	isl_qpolynomial_list *list;

	list = isl_qpolynomial_fold_take_list(fold);
	list = isl_qpolynomial_list_map(list, &realign_domain, r);
	fold = isl_qpolynomial_fold_restore_list(fold, list);

	space = isl_reordering_get_space(r);
	fold = isl_qpolynomial_fold_reset_domain_space(fold, space);

	isl_reordering_free(r);

	return fold;
}

/* isl_qpolynomial_list_map callback for calling
 * isl_qpolynomial_mul_isl_int on "qp".
 */
static __isl_give isl_qpolynomial *mul_int(__isl_take isl_qpolynomial *qp,
	void *user)
{
	isl_int *v = user;

````
- **EN**: This block declares or defines routines around `isl_qpolynomial_realign_domain`, `isl_qpolynomial_fold_realign_domain`, `isl_qpolynomial_fold_take_list`, `isl_qpolynomial_list_map` (+5 more); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `isl_qpolynomial_realign_domain`, `isl_qpolynomial_fold_realign_domain`, `isl_qpolynomial_fold_take_list`, `isl_qpolynomial_list_map` (+5 more) 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理; 保留用于解释意图、用法或算法背景的注释.

### Lines 2064-2089

````c
	qp = isl_qpolynomial_mul_isl_int(qp, *v);
	return qp;
}

__isl_give isl_qpolynomial_fold *isl_qpolynomial_fold_mul_isl_int(
	__isl_take isl_qpolynomial_fold *fold, isl_int v)
{
	isl_qpolynomial_list *list;

	if (isl_int_is_one(v))
		return fold;
	if (fold && isl_int_is_zero(v)) {
		isl_qpolynomial_fold *zero;
		isl_space *space = isl_space_copy(fold->dim);
		zero = isl_qpolynomial_fold_empty(fold->type, space);
		isl_qpolynomial_fold_free(fold);
		return zero;
	}

	fold = isl_qpolynomial_fold_cow(fold);
	if (!fold)
		return NULL;

	if (isl_int_is_neg(v))
		fold->type = isl_fold_type_negate(fold->type);

````
- **EN**: This block declares or defines routines around `isl_qpolynomial_mul_isl_int`, `isl_qpolynomial_fold_mul_isl_int`, `isl_space_copy`, `isl_qpolynomial_fold_empty` (+3 more); contains control flow with 4 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_qpolynomial_mul_isl_int`, `isl_qpolynomial_fold_mul_isl_int`, `isl_space_copy`, `isl_qpolynomial_fold_empty` (+3 more) 相关的例程; 包含控制流结构：4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 2090-2121

````c
	list = isl_qpolynomial_fold_take_list(fold);
	list = isl_qpolynomial_list_map(list, &mul_int, &v);
	fold = isl_qpolynomial_fold_restore_list(fold, list);

	return fold;
}

__isl_give isl_qpolynomial_fold *isl_qpolynomial_fold_scale(
	__isl_take isl_qpolynomial_fold *fold, isl_int v)
{
	return isl_qpolynomial_fold_mul_isl_int(fold, v);
}

/* isl_qpolynomial_list_map callback for calling
 * isl_qpolynomial_scale_val on "qp".
 */
static __isl_give isl_qpolynomial *scale_val(__isl_take isl_qpolynomial *qp,
	void *user)
{
	isl_val *v = user;

	qp = isl_qpolynomial_scale_val(qp, isl_val_copy(v));
	return qp;
}

/* Multiply "fold" by "v".
 */
__isl_give isl_qpolynomial_fold *isl_qpolynomial_fold_scale_val(
	__isl_take isl_qpolynomial_fold *fold, __isl_take isl_val *v)
{
	isl_qpolynomial_list *list;

````
- **EN**: This block declares or defines routines around `isl_qpolynomial_fold_take_list`, `isl_qpolynomial_list_map`, `isl_qpolynomial_fold_restore_list`, `isl_qpolynomial_fold_scale` (+3 more); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `isl_qpolynomial_fold_take_list`, `isl_qpolynomial_list_map`, `isl_qpolynomial_fold_restore_list`, `isl_qpolynomial_fold_scale` (+3 more) 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理; 保留用于解释意图、用法或算法背景的注释.

### Lines 2122-2147

````c
	if (!fold || !v)
		goto error;

	if (isl_val_is_one(v)) {
		isl_val_free(v);
		return fold;
	}
	if (isl_val_is_zero(v)) {
		isl_qpolynomial_fold *zero;
		isl_space *space = isl_qpolynomial_fold_get_domain_space(fold);
		zero = isl_qpolynomial_fold_empty(fold->type, space);
		isl_qpolynomial_fold_free(fold);
		isl_val_free(v);
		return zero;
	}
	if (!isl_val_is_rat(v))
		isl_die(isl_qpolynomial_fold_get_ctx(fold), isl_error_invalid,
			"expecting rational factor", goto error);

	fold = isl_qpolynomial_fold_cow(fold);
	if (!fold)
		goto error;

	if (isl_val_is_neg(v))
		fold->type = isl_fold_type_negate(fold->type);

````
- **EN**: This block declares or defines routines around `isl_val_free`, `isl_qpolynomial_fold_get_domain_space`, `isl_qpolynomial_fold_empty`, `isl_qpolynomial_fold_free` (+3 more); contains control flow with 6 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_val_free`, `isl_qpolynomial_fold_get_domain_space`, `isl_qpolynomial_fold_empty`, `isl_qpolynomial_fold_free` (+3 more) 相关的例程; 包含控制流结构：6 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 2148-2178

````c
	list = isl_qpolynomial_fold_take_list(fold);
	list = isl_qpolynomial_list_map(list, &scale_val, v);
	fold = isl_qpolynomial_fold_restore_list(fold, list);

	isl_val_free(v);
	return fold;
error:
	isl_val_free(v);
	isl_qpolynomial_fold_free(fold);
	return NULL;
}

/* Divide "fold" by "v".
 */
__isl_give isl_qpolynomial_fold *isl_qpolynomial_fold_scale_down_val(
	__isl_take isl_qpolynomial_fold *fold, __isl_take isl_val *v)
{
	if (!fold || !v)
		goto error;

	if (isl_val_is_one(v)) {
		isl_val_free(v);
		return fold;
	}
	if (!isl_val_is_rat(v))
		isl_die(isl_qpolynomial_fold_get_ctx(fold), isl_error_invalid,
			"expecting rational factor", goto error);
	if (isl_val_is_zero(v))
		isl_die(isl_val_get_ctx(v), isl_error_invalid,
			"cannot scale down by zero", goto error);

````
- **EN**: This block declares or defines routines around `isl_qpolynomial_fold_take_list`, `isl_qpolynomial_list_map`, `isl_qpolynomial_fold_restore_list`, `isl_val_free` (+3 more); contains control flow with 4 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_qpolynomial_fold_take_list`, `isl_qpolynomial_list_map`, `isl_qpolynomial_fold_restore_list`, `isl_val_free` (+3 more) 相关的例程; 包含控制流结构：4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 2179-2184

````c
	return isl_qpolynomial_fold_scale_val(fold, isl_val_inv(v));
error:
	isl_val_free(v);
	isl_qpolynomial_fold_free(fold);
	return NULL;
}
````
- **EN**: This block declares or defines routines around `isl_val_free`, `isl_qpolynomial_fold_free`; emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_val_free`, `isl_qpolynomial_fold_free` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

## Key Concepts / 关键概念

- **ISL relations and sets**
  - **CN**: ISL 关系与集合
- **Template/type wrappers**
  - **CN**: 模板与类型包装
- **Loop transformation**
  - **CN**: 循环变换
- **Reduction handling**
  - **CN**: 归约处理
- **Debug and diagnostics**
  - **CN**: 调试与诊断

## Dependencies / 依赖关系

- **System/standard headers**: `isl_map_private.h`, `isl_union_map_private.h`, `isl_polynomial_private.h`, `isl_point_private.h`, `isl_space_private.h`, `isl_lp_private.h`, `isl_seq.h`, `isl_mat_private.h` (+16 more) — Standard-library or system declarations required by the implementation.
  **System/standard headers（CN）**：`isl_map_private.h`, `isl_union_map_private.h`, `isl_polynomial_private.h`, `isl_point_private.h`, `isl_space_private.h`, `isl_lp_private.h`, `isl_seq.h`, `isl_mat_private.h` (+16 more) —— 实现所需的标准库或系统声明。
