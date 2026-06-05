# isl_local.c — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `polly/lib/External/isl/isl_local.c` | `polly/lib/External/isl/isl_local.c` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | Implements or embeds upstream ISL support code used by Polly. | 实现或内嵌 Polly 使用的上游 ISL 支撑代码。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````c
/*
 * Copyright 2011      INRIA Saclay
 * Copyright 2014      Ecole Normale Superieure
 * Copyright 2015      Sven Verdoolaege
 *
 * Use of this software is governed by the MIT license
 *
 * Written by Sven Verdoolaege, INRIA Saclay - Ile-de-France,
 * Parc Club Orsay Universite, ZAC des vignes, 4 rue Jacques Monod,
 * 91893 Orsay, France
 * and Ecole Normale Superieure, 45 rue d'Ulm, 75230 Paris, France
 */

#include <isl/space.h>
#include <isl_vec_private.h>
#include <isl_mat_private.h>
#include <isl_reordering.h>
#include <isl_seq.h>
#include <isl_local_private.h>

````
- **EN**: This block imports ISL, system/standard headers needed by the surrounding code; uses ISL data structures or helpers for polyhedral reasoning; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 引入周边逻辑所需的 ISL、system/standard 头文件; 使用 ISL 数据结构或辅助函数进行多面体推理; 保留用于解释意图、用法或算法背景的注释.

### Lines 21-41

````c
/* Return the isl_ctx to which "local" belongs.
 */
isl_ctx *isl_local_get_ctx(__isl_keep isl_local *local)
{
	if (!local)
		return NULL;

	return isl_mat_get_ctx(local);
}

/* Create an isl_local object from a matrix describing
 * integer divisions.
 *
 * An isl_local object is current defined as exactly such a matrix,
 * so simply return the input.
 */
__isl_give isl_local *isl_local_alloc_from_mat(__isl_take isl_mat *mat)
{
	return mat;
}

````
- **EN**: This block declares or defines routines around `isl_local_get_ctx`, `isl_local_alloc_from_mat`; contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_local_get_ctx`, `isl_local_alloc_from_mat` 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 42-66

````c
/* Return a new reference to "local".
 */
__isl_give isl_local *isl_local_copy(__isl_keep isl_local *local)
{
	return isl_local_alloc_from_mat(isl_mat_copy(local));
}

/* Free "local" and return NULL.
 */
__isl_null isl_local *isl_local_free(__isl_take isl_local *local)
{
	isl_mat_free(local);
	return NULL;
}

/* Return the number of local variables (isl_dim_div),
 * the number of other variables (isl_dim_set) or
 * the total number of variables (isl_dim_all) in "local".
 *
 * Other types do not have any meaning for an isl_local object.
 */
isl_size isl_local_dim(__isl_keep isl_local *local, enum isl_dim_type type)
{
	isl_mat *mat = local;

````
- **EN**: This block declares or defines routines around `isl_local_copy`, `isl_local_free`, `isl_mat_free`, `variables` (+1 more); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `isl_local_copy`, `isl_local_free`, `isl_mat_free`, `variables` (+1 more) 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理; 保留用于解释意图、用法或算法背景的注释.

### Lines 67-89

````c
	if (!local)
		return isl_size_error;
	if (type == isl_dim_div)
		return isl_mat_rows(mat);
	if (type == isl_dim_all) {
		isl_size cols = isl_mat_cols(mat);
		if (cols < 0)
			return isl_size_error;
		return cols - 2;
	}
	if (type == isl_dim_set) {
		isl_size total, n_div;

		total = isl_local_dim(local, isl_dim_all);
		n_div = isl_local_dim(local, isl_dim_div);
		if (total < 0 || n_div < 0)
			return isl_size_error;
		return total - n_div;
	}
	isl_die(isl_local_get_ctx(local), isl_error_unsupported,
		"unsupported dimension type", return isl_size_error);
}

````
- **EN**: This block declares or defines routines around `isl_mat_cols`, `isl_local_dim`, `isl_die`; contains control flow with 6 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_mat_cols`, `isl_local_dim`, `isl_die` 相关的例程; 包含控制流结构：6 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 90-113

````c
#undef TYPE
#define TYPE	isl_local
static
#include "check_type_range_templ.c"

/* Check that "pos" is a valid position for a variable in "local".
 */
static isl_stat isl_local_check_pos(__isl_keep isl_local *local, int pos)
{
	return isl_local_check_range(local, isl_dim_div, pos, 1);
}

/* Given local variables "local",
 * is the variable at position "pos" marked as not having
 * an explicit representation?
 * Note that even if this variable is not marked in this way and therefore
 * does have an explicit representation, this representation may still
 * depend (indirectly) on other local variables that do not
 * have an explicit representation.
 */
isl_bool isl_local_div_is_marked_unknown(__isl_keep isl_local *local, int pos)
{
	isl_mat *mat = local;

````
- **EN**: This block imports system/standard headers needed by the surrounding code; uses preprocessor directives to control compilation, macros, or include guards; defines macros like `TYPE`; declares or defines routines around `isl_local_check_pos`, `depend`, `isl_local_div_is_marked_unknown`; and continues the surrounding implementation details.
- **CN**: 该代码块 引入周边逻辑所需的 system/standard 头文件; 使用预处理指令控制编译、宏或头文件保护; 定义宏，例如 `TYPE`; 声明或定义与 `isl_local_check_pos`, `depend`, `isl_local_div_is_marked_unknown` 相关的例程；并延续周边实现细节。

### Lines 114-135

````c
	if (isl_local_check_pos(local, pos) < 0)
		return isl_bool_error;
	return isl_bool_ok(isl_int_is_zero(mat->row[pos][0]));
}

/* Given local variables "local",
 * does the variable at position "pos" have a complete explicit representation?
 * Having a complete explicit representation requires not only
 * an explicit representation, but also that all local variables
 * that appear in this explicit representation in turn have
 * a complete explicit representation.
 */
isl_bool isl_local_div_is_known(__isl_keep isl_local *local, int pos)
{
	isl_bool marked;
	int i, off;
	isl_size n, cols;
	isl_mat *mat = local;

	if (isl_local_check_pos(local, pos) < 0)
		return isl_bool_error;

````
- **EN**: This block declares or defines routines around `isl_local_div_is_known`; contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_local_div_is_known` 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 136-155

````c
	marked = isl_local_div_is_marked_unknown(local, pos);
	if (marked < 0 || marked)
		return isl_bool_not(marked);

	n = isl_local_dim(local, isl_dim_div);
	cols = isl_mat_cols(mat);
	if (n < 0 || cols < 0)
		return isl_bool_error;
	off = cols - n;

	for (i = n - 1; i >= 0; --i) {
		isl_bool known;

		if (isl_int_is_zero(mat->row[pos][off + i]))
			continue;
		known = isl_local_div_is_known(local, i);
		if (known < 0 || !known)
			return known;
	}

````
- **EN**: This block declares or defines routines around `isl_local_div_is_marked_unknown`, `isl_local_dim`, `isl_mat_cols`, `isl_local_div_is_known`; contains control flow with 1 loop construct(s), 4 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_local_div_is_marked_unknown`, `isl_local_dim`, `isl_mat_cols`, `isl_local_div_is_known` 相关的例程; 包含控制流结构：1 处循环、4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 156-175

````c
	return isl_bool_true;
}

/* Does "local" have an explicit representation for all local variables?
 */
isl_bool isl_local_divs_known(__isl_keep isl_local *local)
{
	int i;
	isl_size n;

	n = isl_local_dim(local, isl_dim_div);
	if (n < 0)
		return isl_bool_error;

	for (i = 0; i < n; ++i) {
		isl_bool unknown = isl_local_div_is_marked_unknown(local, i);
		if (unknown < 0 || unknown)
			return isl_bool_not(unknown);
	}

````
- **EN**: This block declares or defines routines around `isl_local_divs_known`, `isl_local_dim`, `isl_local_div_is_marked_unknown`; contains control flow with 1 loop construct(s), 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_local_divs_known`, `isl_local_dim`, `isl_local_div_is_marked_unknown` 相关的例程; 包含控制流结构：1 处循环、2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 176-198

````c
	return isl_bool_true;
}

/* Compare two sets of local variables, defined over
 * the same space.
 *
 * Return -1 if "local1" is "smaller" than "local2", 1 if "local1" is "greater"
 * than "local2" and 0 if they are equal.
 *
 * The order is fairly arbitrary.  We do "prefer" divs that only involve
 * earlier dimensions in the sense that we consider matrices where
 * the first differing div involves earlier dimensions to be smaller.
 */
int isl_local_cmp(__isl_keep isl_local *local1, __isl_keep isl_local *local2)
{
	int i;
	int cmp;
	isl_bool unknown1, unknown2;
	int last1, last2;
	isl_size n_col;
	isl_mat *mat1 = local1;
	isl_mat *mat2 = local2;

````
- **EN**: This block declares or defines routines around `isl_local_cmp`; emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `isl_local_cmp` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理; 保留用于解释意图、用法或算法背景的注释.

### Lines 199-222

````c
	if (local1 == local2)
		return 0;
	if (!local1)
		return -1;
	if (!local2)
		return 1;

	if (mat1->n_row != mat2->n_row)
		return mat1->n_row - mat2->n_row;

	n_col = isl_mat_cols(mat1);
	if (n_col < 0)
		return -1;
	for (i = 0; i < mat1->n_row; ++i) {
		unknown1 = isl_local_div_is_marked_unknown(local1, i);
		unknown2 = isl_local_div_is_marked_unknown(local2, i);
		if (unknown1 && unknown2)
			continue;
		if (unknown1)
			return 1;
		if (unknown2)
			return -1;
		last1 = isl_seq_last_non_zero(mat1->row[i] + 1, n_col - 1);
		last2 = isl_seq_last_non_zero(mat2->row[i] + 1, n_col - 1);
````
- **EN**: This block declares or defines routines around `isl_mat_cols`, `isl_local_div_is_marked_unknown`, `isl_seq_last_non_zero`; contains control flow with 1 loop construct(s), 8 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_mat_cols`, `isl_local_div_is_marked_unknown`, `isl_seq_last_non_zero` 相关的例程; 包含控制流结构：1 处循环、8 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 223-244

````c
		if (last1 != last2)
			return last1 - last2;
		cmp = isl_seq_cmp(mat1->row[i], mat2->row[i], n_col);
		if (cmp != 0)
			return cmp;
	}

	return 0;
}

/* Return the position of the variables of the given type
 * within the sequence of variables of "local".
 *
 * Only the position of the local variables can be obtained.
 * It is equal to the total number of variables minus
 * the number of local variables.
 */
isl_size isl_local_var_offset(__isl_keep isl_local *local,
	enum isl_dim_type type)
{
	isl_size n_div, n_all;

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_seq_cmp`, `isl_local_var_offset`; contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_seq_cmp`, `isl_local_var_offset` 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 245-270

````c
	if (!local)
		return isl_size_error;
	if (type != isl_dim_div)
		isl_die(isl_local_get_ctx(local), isl_error_unsupported,
			"only the offset of the local variables "
			"can be obtained", return isl_size_error);

	n_div = isl_local_dim(local, isl_dim_div);
	n_all = isl_local_dim(local, isl_dim_all);
	if (n_div < 0 || n_all < 0)
		return isl_size_error;
	return n_all - n_div;
}

/* Reorder the columns of the given local variables according to the
 * given reordering.
 * The order of the local variables themselves is assumed not to change.
 */
__isl_give isl_local *isl_local_reorder(__isl_take isl_local *local,
	__isl_take isl_reordering *r)
{
	isl_mat *div = local;
	int i, j;
	isl_mat *mat;
	int extra;

````
- **EN**: This block declares or defines routines around `isl_die`, `isl_local_dim`, `isl_local_reorder`; contains control flow with 3 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_die`, `isl_local_dim`, `isl_local_reorder` 相关的例程; 包含控制流结构：3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 271-295

````c
	if (!local || !r)
		goto error;

	extra = r->dst_len - r->src_len;
	mat = isl_mat_alloc(div->ctx, div->n_row, div->n_col + extra);
	if (!mat)
		goto error;

	for (i = 0; i < div->n_row; ++i) {
		isl_seq_cpy(mat->row[i], div->row[i], 2);
		isl_seq_clr(mat->row[i] + 2, mat->n_col - 2);
		for (j = 0; j < r->src_len; ++j)
			isl_int_set(mat->row[i][2 + r->pos[j]],
				    div->row[i][2 + j]);
	}

	isl_reordering_free(r);
	isl_local_free(local);
	return isl_local_alloc_from_mat(mat);
error:
	isl_reordering_free(r);
	isl_local_free(local);
	return NULL;
}

````
- **EN**: This block declares or defines routines around `isl_mat_alloc`, `isl_seq_cpy`, `isl_seq_clr`, `isl_int_set` (+2 more); contains control flow with 2 loop construct(s), 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_mat_alloc`, `isl_seq_cpy`, `isl_seq_clr`, `isl_int_set` (+2 more) 相关的例程; 包含控制流结构：2 处循环、2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 296-316

````c
/* Move the "n" variables starting at "src_pos" of "local" to "dst_pos".
 *
 * Moving local variables is not allowed.
 */
__isl_give isl_local *isl_local_move_vars(__isl_take isl_local *local,
	unsigned dst_pos, unsigned src_pos, unsigned n)
{
	isl_mat *mat = local;
	isl_size v_div;

	v_div = isl_local_var_offset(local, isl_dim_div);
	if (v_div < 0)
		return isl_local_free(local);
	if (n == 0)
		return local;

	if (dst_pos >= v_div || src_pos >= v_div)
		isl_die(isl_local_get_ctx(local), isl_error_invalid,
			"cannot move local variables",
			return isl_local_free(local));

````
- **EN**: This block declares or defines routines around `isl_local_move_vars`, `isl_local_var_offset`, `isl_die`; contains control flow with 3 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_local_move_vars`, `isl_local_var_offset`, `isl_die` 相关的例程; 包含控制流结构：3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 317-339

````c
	mat = isl_mat_move_cols(mat, 2 + dst_pos, 2 + src_pos, n);

	return isl_local_alloc_from_mat(mat);
}

/* Does "local" depend on the specified variables?
 *
 * If the specified variables are local variables themselves,
 * then only later local variables could possibly depend on them.
 */
isl_bool isl_local_involves_vars(__isl_keep isl_local *local,
	unsigned first, unsigned n)
{
	isl_mat *mat = local;
	int i, first_div;
	isl_size v_div, n_div;

	v_div = isl_local_var_offset(local, isl_dim_div);
	n_div = isl_local_dim(local, isl_dim_div);
	if (v_div < 0 || n_div < 0 ||
	    isl_local_check_range(local, isl_dim_all, first, n) < 0)
		return isl_bool_error;

````
- **EN**: This block declares or defines routines around `isl_mat_move_cols`, `isl_local_involves_vars`, `isl_local_var_offset`, `isl_local_dim` (+1 more); contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_mat_move_cols`, `isl_local_involves_vars`, `isl_local_var_offset`, `isl_local_dim` (+1 more) 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 340-363

````c
	first_div = (first >= v_div) ? first - v_div + 1 : 0;
	for (i = first_div; i < n_div; ++i) {
		isl_bool unknown;

		unknown = isl_local_div_is_marked_unknown(local, i);
		if (unknown < 0)
			return isl_bool_error;
		if (unknown)
			continue;
		if (isl_seq_any_non_zero(mat->row[i] + 1 + 1 + first, n))
			return isl_bool_true;
	}

	return isl_bool_false;
}

/* Extend a vector "v" representing an integer point
 * in the domain space of "local"
 * to one that also includes values for the local variables.
 * All local variables are required to have an explicit representation.
 * If there are no local variables, then the point is not required
 * to be integral.
 */
__isl_give isl_vec *isl_local_extend_point_vec(__isl_keep isl_local *local,
````
- **EN**: This block declares or defines routines around `isl_local_div_is_marked_unknown`, `isl_local_extend_point_vec`; contains control flow with 1 loop construct(s), 3 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_local_div_is_marked_unknown`, `isl_local_extend_point_vec` 相关的例程; 包含控制流结构：1 处循环、3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 364-387

````c
	__isl_take isl_vec *v)
{
	isl_size dim, n_div, size;
	isl_bool known;
	isl_mat *mat = local;

	if (!local || !v)
		return isl_vec_free(v);
	known = isl_local_divs_known(local);
	if (known < 0)
		return isl_vec_free(v);
	if (!known)
		isl_die(isl_local_get_ctx(local), isl_error_invalid,
			"unknown local variables", return isl_vec_free(v));
	dim = isl_local_dim(local, isl_dim_set);
	n_div = isl_local_dim(local, isl_dim_div);
	size = isl_vec_size(v);
	if (dim < 0 || n_div < 0 || size < 0)
		return isl_vec_free(v);
	if (size != 1 + dim)
		isl_die(isl_local_get_ctx(local), isl_error_invalid,
			"incorrect size", return isl_vec_free(v));
	if (n_div == 0)
		return v;
````
- **EN**: This block declares or defines routines around `isl_local_divs_known`, `isl_die`, `isl_vec_free`, `isl_local_dim` (+1 more); contains control flow with 6 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_local_divs_known`, `isl_die`, `isl_vec_free`, `isl_local_dim` (+1 more) 相关的例程; 包含控制流结构：6 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 388-406

````c
	if (!isl_int_is_one(v->el[0]))
		isl_die(isl_local_get_ctx(local), isl_error_invalid,
			"expecting integer point", return isl_vec_free(v));
	{
		int i;
		v = isl_vec_add_els(v, n_div);
		if (!v)
			return NULL;

		for (i = 0; i < n_div; ++i) {
			isl_seq_inner_product(mat->row[i] + 1, v->el,
						1 + dim + i, &v->el[1+dim+i]);
			isl_int_fdiv_q(v->el[1+dim+i], v->el[1+dim+i],
					mat->row[i][0]);
		}
	}

	return v;
}
````
- **EN**: This block declares or defines routines around `isl_die`, `isl_vec_free`, `isl_vec_add_els`, `isl_seq_inner_product` (+1 more); contains control flow with 1 loop construct(s), 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_die`, `isl_vec_free`, `isl_vec_add_els`, `isl_seq_inner_product` (+1 more) 相关的例程; 包含控制流结构：1 处循环、2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

## Key Concepts / 关键概念

- **ISL relations and sets**
  - **CN**: ISL 关系与集合
- **Loop transformation**
  - **CN**: 循环变换

## Dependencies / 依赖关系

- **ISL headers**: `isl/space.h` — ISL relation/set primitives that support polyhedral analysis and transformations.
  **ISL headers（CN）**：`isl/space.h` —— 支撑多面体分析与变换的 ISL 关系/集合原语。
- **System/standard headers**: `isl_vec_private.h`, `isl_mat_private.h`, `isl_reordering.h`, `isl_seq.h`, `isl_local_private.h`, `check_type_range_templ.c` — Standard-library or system declarations required by the implementation.
  **System/standard headers（CN）**：`isl_vec_private.h`, `isl_mat_private.h`, `isl_reordering.h`, `isl_seq.h`, `isl_local_private.h`, `check_type_range_templ.c` —— 实现所需的标准库或系统声明。
